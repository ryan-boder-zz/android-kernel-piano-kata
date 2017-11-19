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

Sound waves are represented in these samples using [pulse code modulation](https://en.wikipedia.org/wiki/Pulse-code_modulation) (PCM) which is a fancy way of saying each sample represents the amplitude of sound at that moment in time. The time of the first sample (in each channel) is t = 0 seconds. Since the goldfish audio device operates at 44.1 kHz the time of the second sample is t = 1/44100 seconds and the third sample is t = 2/44100 seconds, etc...

The difference between notes on a piano is the frequency at which those samples oscillate up and down. Refer to [this chart](https://en.wikipedia.org/wiki/Piano_key_frequencies) for the frequency of each note.

#### Generate notes with square, saw, sine waves

Program the piano driver so that when you press a key listed on one of the 3 right columns in the table your driver generates and plays the appropriate wave of the corresponding frequency for 1 second at a fixed volume.

| Note | Freq (Hz) | Square | Saw | Sine |
| ---- | --------- | ------ | --- | ---- |
| a    | 440.000   | /      | ;   | p    |
| g#   | 415.305   | .      | l   | o    |
| g    | 391.995   | ,      | k   | i    |
| f#   | 369.994   | m      | j   | u    |
| f    | 349.228   | n      | h   | y    |
| e    | 329.628   | b      | g   | t    |
| d#   | 311.127   | v      | f   | r    |
| d    | 293.665   | c      | d   | e    |
| c#   | 277.183   | x      | s   | w    |
| c    | 261.626   | z      | a   | q    |

Note that the kernel has no math library functions and doesn't even support floating point operations! How will you approximate a sine wave?

#### Fade out sound effect

Enhance the piano driver so that when you first press a key the note is played loudly but it trails off and fades out like a real piano key.
