## Interfacing with the Goldfish audio device

The goldfish virtual audio device is documented [here](https://android.googlesource.com/platform/external/qemu/+/master/docs/GOLDFISH-VIRTUAL-HARDWARE.TXT) in section VI. Refer to this document as needed for details on how to interface with the device.

#### Sharing data between calling contexts

Your device driver code will be called in multiple contexts:

1. Driver lifecycle functions such as probe and remove
1. User space processes calling your file operation functions
1. Hardware interrupts calling your interrupt handler

Create a data structure for sharing data between these calling contexts.

```c
struct goldfish_piano {

};
static struct goldfish_piano *piano_data;
```

In a production driver best practice is to [manage these objects dynamically](https://static.lwn.net/images/pdf/LDD3/ch03.pdf) so your driver can support multiple device instances but for simplicity's sake we will exploit the fact that we know the emulator only has one audio device declare a static pointer `piano_data` to keep track of the single instance.

:white_check_mark: Update your probe function to allocate memory _in kernel space_ for this `struct goldfish_piano` instance and set `piano_data` to point at this object. Update your remove function free this memory.

#### Map the device physical address to virtual memory

:white_check_mark: In your probe function use [ioremap()](http://learnlinuxconcepts.blogspot.com/2014/10/what-is-ioremap.html) to [get a virtual memory page](https://lwn.net/Articles/653585) (of PAGE_SIZE) that you can read and write to interface with the device. Store this virtual address in your `goldfish_piano` instance as the base address for interacting with the audio device.

:white_check_mark: In order to call ioremap() you need to know the base physical address of the device IO registers. Use [platform_get_resource()](https://lwn.net/Articles/448499) to get the `struct resource` which has the physical address in it's `start` field. The audio device is the first device of this type in the goldfish virtual hardware.

:white_check_mark: Don't forget to iounmap() the virtual address in your remove function.
