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

:white_check_mark: Don't forget to iounmap() the virtual memory when your driver is removed.

#### Allocate coherent memory for DMA

The goldfish audio device uses [direct memory access](https://en.wikipedia.org/wiki/Direct_memory_access) (DMA). Devices that require passing a lot of data between the device and the CPU typically use DMA so the CPU doesn't have to stay busy dealing with each piece of data one by one as it's being produced or consumed.

DMA devices need [memory coherence](https://en.wikipedia.org/wiki/Memory_coherence) with the CPU to ensure that changes are immediately visible to both sides and not stuck somewhere in cache.

:white_check_mark: Use [dma_alloc_coherent()](https://www.kernel.org/doc/Documentation/DMA-API.txt) to allocate a contiguous region of coherent memory in kernel space that can be used as shared memory with the audio device. The region should be big enough to contain 2 x 16384 byte write buffers which is all the memory you need shared with the device.

:white_check_mark: Don't forget to dma_free_coherent() the DMA memory when your driver is removed.

#### Set up interrupt handling

When the audio device wants to notify the CPU that something has occurred and needs attention it interrupts the CPU. We need to set up an interrupt handler that will be called when the audio device interrupts the CPU.

:white_check_mark: Create an [interrupt handler function](https://notes.shichao.io/lkd/ch7/#writing-an-interrupt-handler) that just logs that an interrupt was received and acknowledges it as handled.

```c
#include <linux/interrupt.h>

static irqreturn_t goldfish_piano_interrupt(int irq, void *dev)
{
    printk("goldfish_piano_interrupt\n");
    return IRQ_HANDLED;
}
```

:white_check_mark: Use [request_irq()](https://notes.shichao.io/lkd/ch7/#registering-an-interrupt-handler) to register your interrupt handler as a shared handler with the kernel so that it will be called when the audio device interrupts the CPU.

:white_check_mark: In order to call request_irq() you need to know the interrupt line number that the audio device will use to interrupt the CPU. Use [platform_get_irq()](https://lwn.net/Articles/448499) to get the interrupt line number for the goldfish audio device.

:white_check_mark: Don't forget to free_irq() the IRQ line when your driver is removed.

#### Tell the device where to find the write buffers

You previously allocated a DMA memory region big enough for 2 write buffers but the goldfish audio device doesn't know where to find those buffers.

:white_check_mark: Use [writel()](http://www.xml.com/ldd/chapter/book/ch08.html#t4) to set both kernel output buffer addresses in the SET_WRITE_BUFFER_1 & SET_WRITE_BUFFER_2 I/O registers.

#### Check status when an interrupt is received

As a shared interrupt handler, your interrupt handler needs to check the status of the audio device to determine whether the interrupt actually came from the audio device.

:white_check_mark: Use [readl()](http://www.xml.com/ldd/chapter/book/ch08.html#t4) to get the interrupt status from the audio device's INT_STATUS I/O register. Note that you only care about the bottom 3 bits in the INT_STATUS register. If the interrupt did come from the audio device you should handle it appropriately and return `IRQ_HANDLED`. If the interrupt did not come from the audio device you should do nothing and return `IRQ_NONE`.

#### Using the write buffers

The goldfish audio device uses 2 write buffers for [double buffering](https://en.wikipedia.org/wiki/Multiple_buffering) which allows the device to be reading data from one buffer while we are writing data to the other. We can have the second buffer ready with data and waiting so the device can near-instantly switch to it when it finishes "playing" all the data in the first buffer. This way the sound keeps playing continuously without a break when the device switches buffers.

The audio device plays 44.1 kHz stereo (2 channel) sound with 16 bit samples. It uses 2x16 = 32 bits = 4 bytes every 1/44100 seconds. Our 16384 byte write buffers can only hold 16384/4 = 4096 samples so a write buffer can only hold enough data to play sound for 4096/44100 = 0.09287 seconds. That's just under 1/10 of a second so to play sound for 1 second we will have to switch buffers 10 times!

When you are writing data to the audio device you will need to keep track of which buffer is currently being consumed by the device and which buffer is available for you to produce data into.

To produce data into buffer 1 you will need to wait for the device to tell you that buffer 1 is available by reading the status register in your interrupt handler and checking whether the bit 0 is 1. Then you can produce data into buffer 1 and use [writel()](http://www.xml.com/ldd/chapter/book/ch08.html#t4) to write the AUDIO_WRITE_BUFFER_1 register with the number of bytes you have put into buffer 1. Use the same approach for buffer 2 but check bit 1 in the status register and write the byte count to the AUDIO_WRITE_BUFFER_2 register.

The difficult part is that you should not produce all that data into write buffers directly in the interrupt handler. Your interrupt handler should do minimal work and be extremely fast. It also [executes without a process context](https://notes.shichao.io/lkd/ch7/#difference-from-the-process-context) which limits what you can do. Instead your interrupt handler should just check the status register and then wake up a waiting process (hint: your write function) to do the actual work. Consider using a [wait queue](http://tuxthink.blogspot.com/2011/04/wait-queues.html) to put a process to sleep and then waking up that process in your interrupt handler if there is work to be done.

#### Synchronization

In order to do minimal work in the interrupt handler and defer the heavy work to a process you may need to share data between these 2 potentially concurrent threads of execution. Therefore you need synchronization. Consider using a [spinlock](http://www.linuxjournal.com/article/5833) (with irqsave) to synchronize access to shared data and avoid a race condition.

#### Enabling and disabling interrupts

It is good practice to only enable interrupts when the device is actually in use.

:white_check_mark: Use [writel()](http://www.xml.com/ldd/chapter/book/ch08.html#t4) to write the AUDIO_INT_ENABLE register enabling interrupts for both write buffers when `/dev/piano` is opened and disabling all interrupts from the audio device when `/dev/piano` is closed.
