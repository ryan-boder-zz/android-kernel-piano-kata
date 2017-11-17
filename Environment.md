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
```
host$ cd ~/Library/Android/sdk
host$ emulator/emulator @Pixel_API_26
```

#### Verify ADB access

Verify that you can get a shell on your running AVD via ADB. You will need to run `adb root` after each time you boot the AVD before you can use `adb shell`.

OSX:
```
host$ cd ~/Library/Android/sdk
host$ platform-tools/adb root
host$ platform-tools/adb shell
```

Going forward we will simply write use `adb` when referring to this executable in `platform-tools/adb` of your Android SDK.

## Set up your kernel build environment

The easiest environment to build Linux on is Linux. Use this [VirtualBox Vagrant](https://github.com/ryan-boder/aosp-dev-box) to set up a basic Ubuntu VM for building the kernel.

#### Clone the Android emulator kernel source

In the home directory of your VM do
```
vagrant$ git clone https://android.googlesource.com/kernel/goldfish
vagrant$ cd goldfish
vagrant$ git checkout android-goldfish-3.18
```

> Linux file systems are case-sensitive by default but OSX and Windows are not. If you share a folder from your OSX or Windows host to your Linux VM it will be mounted in Linux as a case-insensitive file system. The Linux kernel must be built on a case-sensitive file system so you can not built the kernel in a host shared folder unless you have made that folder case-sensitive on your host.

#### Set your toolchain and target architecture

Either put this in `.bashrc` or do this every time your log into the VM.
```
vagrant$ export ARCH=x86
vagrant$ export CROSS_COMPILE=/opt/android/ndk/android-ndk-r15c/toolchains/x86-4.9/prebuilt/linux-x86_64/bin/i686-linux-android-
```

#### Compile the Linux kernel

The first time you compile it will take a while. After that you can build pretty quickly with just `make`.
```
vagrant$ make i386_ranchu_defconfig
vagrant$ make
...
...
...
Kernel: arch/x86/boot/bzImage is ready
```

#### Copy the kernel to the host

Copy the kernel image from your VM to your host so that you can boot it on the emulator.
```
vagrant$ cp arch/x86/boot/bzImage /vagrant/share/
```

#### Boot the kernel in the AVD

OSX:
```
host$ cd ~/Library/Android/sdk
host$ emulator/emulator -kernel ~/aosp-dev-box/share/bzImage -show-kernel @Pixel_API_26
```

## Verify emulator audio

Verify that audio works in your AVD. Copy the Guitar.wav to your AVD and play it using the kernel's emulator audio driver.

```
host$ adb push Guitar.wav /sdcard
host$ adb shell
android# cat /sdcard/Guitar.wav > /dev/eac
```

If you hear the file play then your emulator's audio device works.
