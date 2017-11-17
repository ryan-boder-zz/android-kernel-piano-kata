# Android Linux Kernel Piano Kata

In this kata you will build a piano driver in the Linux kernel for the x86 Android emulator.

## Set Up

#### Development environment

Use the [set up instructions](Setup.md) to set up your development environment and tools.

#### Replace the included audio driver

The goldfish emulator kernel includes a driver for the audio device located at `drivers/staging/goldfish/goldfish_audio.c`. We're going to replace it with our own driver.

```
vagrant$ $EDITOR drivers/staging/goldfish/Makefile
```

Replace `obj-$(CONFIG_GOLDFISH_AUDIO) += goldfish_audio.o` with `obj-n += goldfish_audio.o` to disable the included audio driver and then add a line that says `obj-y += goldfish_piano.o` to enable our own driver.

```
obj-n += goldfish_audio.o
obj-y += goldfish_piano.o
```

## Verified versions

- Android Studio 3.0
- VirtualBox 5.2.0
- Vagrant 2.0.1
