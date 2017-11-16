## Set up the Android emulator

#### Install Android Studio

Install [Android Studio](https://developer.android.com/studio/index.html) which includes the Android emulator.

#### Create the Android Virtual Device

In Android Studio go to Tools -> Android -> AVD Manager and choose Create Virtual Device...

1. Choose "Pixel" as the device definition.
1. Choose "Oreo" (API Level 26, ABI x86) as the system image. You may need to click the Download link.
1. Verify your configuration. Going forward we'll assume your AVD name is "Pixel API 26".
1. Select your new AVD and hit the play button (right arrow) to launch it and verify it works.
1. Close the AVD when finished.

#### Launch the AVD from the command line

In order to boot your own kernel you will need to launch the emulator from the command line. Verify that this works.

OSX:
```bash
$ cd ~/Library/Android/sdk
$ emulator/emulator @Pixel_API_26
```

#### Verify ADB access

Verify that you can get a shell on your running AVD via ADB. You will need to run `adb root` after each time you boot the AVD before you can use `adb shell`.

OSX:
```bash
$ cd ~/Library/Android/sdk
$ platform-tools/adb root
$ platform-tools/adb shell
```

## Set up your kernel build environment

The easiest environment to build Linux on is Linux. Use this [VirtualBox Vagrant](https://github.com/ryan-boder/aosp-dev-box) to set up a basic Ubuntu VM for building the kernel.

#### Clone the Android emulator kernel source

In the home directory of your VM do
```bash
$ git clone https://android.googlesource.com/kernel/goldfish
$ cd goldfish
```

#### Set your toolchain and target architecture

Either put this in `.bashrc` or do this every time your log into the VM.
```bash
$ export ARCH=x86
$ export CROSS_COMPILE=/opt/android/ndk/android-ndk-r15c/toolchains/x86-4.9/prebuilt/linux-x86_64/bin/i686-linux-android-
```

#### Compile the Linux kernel

The first time you compile it will take a while. After that you can build pretty quickly with just `make`.
```bash
$ make i386_ranchu_defconfig
$ make
...
...
...
Kernel: arch/x86/boot/bzImage is ready
```

#### Copy the kernel to the host

Copy the kernel image from your VM to your host so that you can boot it on the emulator.
```bash
$ cp arch/x86/boot/bzImage /vagrant/share/
```

#### Boot the kernel in the AVD

OSX:
```bash
$ cd ~/Library/Android/sdk
$ emulator/emulator -kernel ~/aosp-dev-box/share/bzImage -show-kernel @Pixel_API_26
```
