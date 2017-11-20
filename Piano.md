## Implement the piano device driver

The piano device works in conjunction with a script to capture key presses on your keyboard and immediately pass them to your device driver by writing them to `/dev/piano`.

You will turn your keyboard into a piano that can play the notes in [C major](https://en.wikipedia.org/wiki/C_major) as [square, sawtooth and sine waves](https://www.youtube.com/watch?v=j2uB4nKzGlg).

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

Sound waves are represented in these samples using [pulse code modulation](https://en.wikipedia.org/wiki/Pulse-code_modulation) (PCM) which is a fancy way of saying each sample represents the amplitude of sound at that moment in time. The time of the first sample (in each channel) is t = 0 seconds. Since the goldfish audio device operates at 44.1 kHz the time of the second sample is t = 1/44100 seconds and the third sample is t = 2/44100 seconds, etc...

The difference between notes on a piano is the frequency at which those samples oscillate up and down.

#### Generate the notes in C major as square waves

Program the piano driver so that when you press a keyboard key in the table below your driver generates and plays a square wave of the corresponding frequency for 1 second at a fixed volume.

The bottom row of your keyboard will [play the notes](https://www.youtube.com/watch?v=tHWAo9A4YdY) in C major. The middle row on your keyboard will play the sharp notes on the black keys in between.

| Note | Freq (Hz) | Keyboard |
| ---- | --------- | -------- |
| c    | 523.251   | ,        |
| b    | 493.883   | m        |
| a#   | 466.164   | j        |
| a    | 440.000   | n        |
| g#   | 415.305   | h        |
| g    | 391.995   | b        |
| f#   | 369.994   | g        |
| f    | 349.228   | v        |
| e    | 329.628   | c        |
| d#   | 311.127   | d        |
| d    | 293.665   | x        |
| c#   | 277.183   | s        |
| c    | 261.626   | z        |

#### Generate the notes as sawtooth and sine waves

Now that you have a working piano that plays square waves, extend it to support sawtooth and sine wave modes.

Program the piano driver so that when you press '1', '2' & '3' the driver will switch between square, sawtooth and sine wave modes respectively.

| Mode | Keyboard |
| ---- | -------- |
| switch to square wave mode | 1 |
| switch to sawtooth wave mode | 2 |
| switch to sine wave mode | 3 |

Note that the kernel has no math library functions and doesn't even support floating point operations! How will you approximate a sine wave?

#### Fade out sound effect

Enhance the piano driver so that when you first press a key the note is played loudly but it trails off and fades out like a real piano key.
