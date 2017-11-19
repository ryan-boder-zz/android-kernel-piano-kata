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

#### Playing sound in the audio device

The goldfish audio device takes a stream of pairs of 16 bit signed integer samples. The first channel is the first 16 bit integer in each pair, the second channel is the second integer in each pair.

You can operate on the audio data in C code by setting a `short*` to point at the beginning of a write buffer. For example:

```c
short *samples = (short*) piano_data->write_buffer_1;
samples[0] = ...; // First sample in channel 1
samples[1] = ...; // First sample in channel 2
samples[2] = ...; // Second sample in channel 1
samples[3] = ...; // Second sample in channel 2
```

Sound waves are represented in these samples using [pulse code modulation](https://en.wikipedia.org/wiki/Pulse-code_modulation) (PCM) which is a fancy way of saying each sample represents the amplitude (or volume) of sound at that moment in time. The time of the first sample (in each channel) is t = 0 seconds. Since the goldfish audio device operates at 44.1 kHz the time of the second sample is t = 1/44100 seconds and the third sample is t = 2/44100 seconds, etc...
