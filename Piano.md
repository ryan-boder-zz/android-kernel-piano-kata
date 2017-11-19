## Implement the piano device driver

The piano device works in conjunction with a script to capture key presses on your keyboard and immediately pass them to your device driver by writing them to `/dev/piano`.

#### The piano script

Push the piano script to `/sdcard` on your AVD.

```
host$ adb push piano /sdcard/
host$ adb shell
android# sh /sdcard/piano
```

Hit a few keys and watch kernel log to see what functions are called in your driver. This piano script serves as the input mechanism to control the piano device.
