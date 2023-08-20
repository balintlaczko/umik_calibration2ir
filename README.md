# UMIK Calibration to correction IR
A minimal app built with [Max](https://cycling74.com/products/max) for generating a calibration impulse response wav for miniDSP [UMIK-1](https://www.minidsp.com/products/acoustic-measurement/umik-1) or [UMIK-2](https://www.minidsp.com/products/acoustic-measurement/umik-2) measurement microphone based on a calibration txt file.

![](https://github.com/balintlaczko/umik_calibration2ir/blob/main/ui_example_loaded.png)

## Motivation
While I appreciate the integration of my UMIK-2 in [REW](https://www.roomeqwizard.com/) I wanted to have the mic's calibration file not only as a `.txt` but also as a correction impulse response `.wav` file, so I can apply the correction to the mic feed or recordings with any convolver tool or plugin available in modern audio software.

## Usage
Download the latest release. At the moment there are ready builds for _windows-x86-64_ and for _macos-arm64_, but since the source is a Max patch, you can also just opt for that one (if you have Max installed). Note that if you use the patch in Max, you might have to install the [HISS Tools](https://www.thehiss.org/) package from the Package Manager -- although the externals should be included in the project too.

If the app or patch is open, drop the calibration `.txt` file (that comes with your UMIK-1 or UMIK-2) into the rectangle in the top left corner, then set your desired parameters for the correction impulse response. The result should update every time you change a setting. When you are ready, hit 'Export wav' and choose a location for the `.wav` file. That's it, now you can load this in any convolver plugin or other tool to apply the correction to the mic feed or a recording.

## How it works
This is for documenting what exactly happens under the hood. I am not a trained acoustician, so if you see anything that looks wrong or suggest additional steps, let me know in an Issue on this repo.

### Calibration files
As far as I have seen UMIK calibration files usually start with a few lines in quotes, describing the Sens Factor, the serial number of the mic and other info. After this "header" all the lines are pairs of frequency and gain values. There isn't a load of documentation about calibration files online, so most of what I know is based on [REW's documentation about calibration files](https://www.roomeqwizard.com/help/help_en-GB/html/calfiles.html). There, in the __Calibration File Format__ section, it says
> _It should contain the actual gain (and optionally phase) response of the meter or microphone at the frequencies given, these will then be subtracted from subsequent measurements._

Which, in my understanding, means that the listed gain values are _not_ what should be applied as a correction, but what the microphone actually produces at a given frequency. In other words, the correction should be an inverse of the curve described by the gain values. The app/patch is currently based on this premise, so let me know if you suspect I'm wrong.

### 1. Read and parse .txt

First the program reads the .txt files and parses each line. Lines that are in quotes are ignored, and the rest are assumed to be pairs of frequency and gain values. The gains sometimes use scientific notation in the `.txt` so these are parsed into floats.

### 2. Generate the correction IR

The parsed list of frequencies and gains are fed into an `iruser~` that generates the response IR with linear phase and the choses FFT size and sample rate. This is what shows up blue on the graph. It is inverted by `irinvert~`, then we generate a minimum phase version of that inverted IR via `irphase~`. Finally, we crop the result with `irtrimnorm~` using the length, fade in and fade out parameters. (As far as I know all fades are linear.) This cropped version is the final IR that we can export, and which shows up red on the graph. The exported `.wav` file will be 32-bit float.
