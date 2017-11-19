## Create a skeleton piano device driver

#### The Goldfish audio device

The goldfish virtual audio device is documented [here](https://android.googlesource.com/platform/external/qemu/+/master/docs/GOLDFISH-VIRTUAL-HARDWARE.TXT) in section VI. Refer to this document as needed for details on how to interface with the device.

#### Replace the audio driver in the build

The goldfish emulator kernel includes a driver for the audio device located at `drivers/staging/goldfish/goldfish_audio.c`. We're going to replace it with our own driver.

```
vagrant$ $EDITOR drivers/staging/goldfish/Makefile
```

Replace `obj-$(CONFIG_GOLDFISH_AUDIO) += goldfish_audio.o` with `obj-n += goldfish_audio.o` to disable the included audio driver and then add a line that says `obj-y += goldfish_piano.o` to enable our own driver.

```
obj-n += goldfish_audio.o
obj-y += goldfish_piano.o
```

#### Create a platform driver for the device

```
vagrant$ $EDITOR drivers/staging/goldfish/goldfish_piano.c
```

```c
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/acpi.h>

static int goldfish_piano_probe(struct platform_device *pdev)
{
    printk("goldfish_piano_probe\n");
    return 0;
}

static int goldfish_piano_remove(struct platform_device *pdev)
{
    printk("goldfish_piano_remove\n");
    return 0;
}

static const struct acpi_device_id goldfish_piano_acpi_match[] = {
    { "GFSH0005", 0 },
    {}
};
MODULE_DEVICE_TABLE(acpi, goldfish_piano_acpi_match);

static struct platform_driver goldfish_piano_driver = {
    .probe = goldfish_piano_probe,
    .remove = goldfish_piano_remove,
    .driver = {
        .name = "goldfish_piano",
        .acpi_match_table = ACPI_PTR(goldfish_piano_acpi_match)
  }
};

module_platform_driver(goldfish_piano_driver);
```

Now rebuild the kernel with your new device driver, copy it to your host and boot it up in the AVD.

```
vagrant$ make
vagrant$ cp arch/x86/boot/bzImage /vagrant/share/
host$ emulator -kernel ~/aosp-dev-box/share/bzImage -show-kernel @Pixel_API_26
```

You should see "goldfish_piano_probe" printed in the kernel log as the kernel boots up.

#### Create a character device to communicate with user space

Add the following to your driver to create a characters device accessible user space programs called `/dev/piano`.

```c
#include <linux/miscdevice.h>

static ssize_t goldfish_piano_read(struct file *fp, char __user *buf, size_t count, loff_t *pos)
{
    char* hello = "Hello World\n";
    printk("goldfish_piano_read: count=%d pos=%d\n", count, (int) *pos);

    if (0 == *pos) {
        *pos = strlen(hello);
        memcpy(buf, hello, *pos);
        return *pos;
    } else {
        return 0;
    }
}

static ssize_t goldfish_piano_write(struct file *fp, const char __user *buf, size_t count, loff_t *pos)
{
    printk("goldfish_piano_write: count=%d buf=%.*s\n", count, count, buf);
    return count;
}

static int goldfish_piano_open(struct inode *ip, struct file *fp)
{
    printk("goldfish_piano_open\n");
    return 0;
}

static int goldfish_piano_release(struct inode *ip, struct file *fp)
{
    printk("goldfish_piano_release\n");
    return 0;
}

static const struct file_operations goldfish_piano_fops = {
    .owner = THIS_MODULE,
    .read = goldfish_piano_read,
    .write = goldfish_piano_write,
    .open = goldfish_piano_open,
    .release = goldfish_piano_release,
};

static struct miscdevice goldfish_piano_device = {
    .minor = MISC_DYNAMIC_MINOR,
    .name = "piano",
    .fops = &goldfish_piano_fops,
};
```

Update your probe and remove functions to register the character device.

```c
static int goldfish_piano_probe(struct platform_device *pdev)
{
    int ret;
    printk("goldfish_piano_probe\n");

    ret = misc_register(&goldfish_piano_device);
    if (ret) {
        dev_err(&pdev->dev, "misc_register returned %d in goldfish_piano_probe\n", ret);
        return ret;
    }

    return 0;
}

static int goldfish_piano_remove(struct platform_device *pdev)
{
    printk("goldfish_piano_remove\n");
    misc_deregister(&goldfish_piano_device);
    return 0;
}
```

Rebuild the kernel and boot it up. Then ADB in to interact with `/dev/piano`.

```
android# cat /dev/piano
Hello World
android# echo -n "Hello World" > /dev/piano
```

You should see your open, read, write and close functions being called in the kernel log.
