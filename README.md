# Android Linux Kernel Piano Kata

In this kata you will build a piano driver in the Linux kernel for the x86 Android emulator.

Thorough error handling and graceful degradation when things go wrong is important. While this is generally true it's especially true in kernel programming. If your code crashes, leaves the system in a bad state or leaks resources you will bring down the whole system, not just your app.

## Set up the development environment

Use the [environment set up guide](Environment.md) to set up your development environment and tools.

## Create a skeleton piano device driver

Create a [skeleton piano device driver](Skeleton.md) that we will build our piano on.

## Interface with the Goldfish audio device

Interface with the [Goldfish emulated audio device](Hardware.md) to play audio on the Android emulator.

## Verified versions

- Android Studio 3.0
- VirtualBox 5.2.0
- Vagrant 2.0.1
