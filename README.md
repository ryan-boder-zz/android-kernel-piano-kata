# Android Linux Kernel Piano Kata

In this kata you will build a piano driver in the Linux kernel for the x86 Android emulator.

## Set up the development environment

Use the [environment set up guide](Environment.md) to set up your development environment and tools.

## Create a skeleton piano device driver

Create a [skeleton piano device driver](Skeleton.md) that we will build our piano on.

## The Goldfish audio device

The goldfish virtual audio device is documented [here](https://android.googlesource.com/platform/external/qemu/+/master/docs/GOLDFISH-VIRTUAL-HARDWARE.TXT) in section VI. Refer to this document as needed for details on how to interface with the device.

## Verified versions

- Android Studio 3.0
- VirtualBox 5.2.0
- Vagrant 2.0.1
