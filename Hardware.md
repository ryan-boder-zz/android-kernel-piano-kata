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

Like user space, the kernel operates with virtual memory but the audio device is located at a specific physical memory address. We need to map some virtual memory to that physical memory region in order to talk to the device.

:white_check_mark: In your probe function use [ioremap()](http://learnlinuxconcepts.blogspot.com/2014/10/what-is-ioremap.html) to [get a virtual memory page](https://lwn.net/Articles/653585) (of PAGE_SIZE) that you can read and write to interface with the device. Store this virtual address in your `goldfish_piano` instance as the base address for interacting with the audio device.

:white_check_mark: In order to call ioremap() you need to know the base physical address of the device IO registers. Use [platform_get_resource()](https://lwn.net/Articles/448499) to get the `struct resource` which has the physical address in it's `start` field. The audio device is the first device of this type in the goldfish virtual hardware.

:white_check_mark: Don't forget to iounmap() the virtual memory in your remove function.

#### Allocate coherent memory for DMA

The goldfish audio device uses [direct memory access](https://en.wikipedia.org/wiki/Direct_memory_access) (DMA). Devices that require passing a lot of data between the device and the CPU typically use DMA so the CPU doesn't have to stay busy dealing with each piece of data one by one as it's being produced or consumed.

DMA devices need [memory coherence](https://en.wikipedia.org/wiki/Memory_coherence) with the CPU to ensure that changes are immediately visible to both sides and not stuck somewhere in cache.

:white_check_mark: Use [dma_alloc_coherent()](https://www.kernel.org/doc/Documentation/DMA-API.txt) to allocate a contiguous region of coherent memory in kernel space that can be used as shared memory with the audio device. The region should be big enough to contain all the memory you need shared with the device.

:white_check_mark: Don't forget to dma_free_coherent() the DMA memory when you're done.
