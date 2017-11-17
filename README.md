# Android Linux Kernel Piano Kata

In this kata you will build a piano driver in the Linux kernel for the x86 Android emulator.

## Set Up

#### Development environment

Use the [set up instructions](Setup.md) to set up your development environment and tools.

## Create the piano device driver

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

#### Create the new driver file

```
vagrant$ $EDITOR drivers/staging/goldfish/goldfish_piano.c
```

```c
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/acpi.h>

static int goldfish_piano_probe(struct platform_device *pdev)
{
        printk("goldfish_piano_probe");
        return 0;
}

static int goldfish_piano_remove(struct platform_device *pdev)
{
        printk("goldfish_piano_remove");
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

## Verified versions

- Android Studio 3.0
- VirtualBox 5.2.0
- Vagrant 2.0.1
