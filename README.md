# screendump / Elektron-Screen-Tool

See your Elektron instrument's screen in a window easily and quickly giving you a source to record for video making.
<img width="512" height="256" alt="screendump-20261018-223646" src="https://github.com/user-attachments/assets/80953731-1989-4879-9edf-70e444fa57ab" />



It uses the instrument's own screenshot command over MIDI. **Nothing on the
instrument is modified** — no firmware change, no flashing.

Works on macOS and Linux. Windows awaits a tester.

> **Credit:** the capture tool is [Jakob Bak's
> screendump](https://github.com/jakobbak/screendump), MIT licensed — see
> `LICENSE`, which stays with the copyright intact. This fork adds a window for
> recording the screen in OBS, and support for the Digitakt II. The wire format,
> the renderings, the recording and the video export are his work.






GUI Example:

<img width="637" height="395" alt="image" src="https://github.com/user-attachments/assets/3919b82a-0fa6-42c1-83cf-c921e17b14ba" />

## Run it

`screendump` is one Python file with no installer, so every `$ screendump` in
this README is shorthand for *that file, invoked somehow*. Nothing puts a
command on your `PATH` for you unless you ask it to. Two ways, pick either.

**Link the file onto your `PATH`**, which is what makes the examples below copy
and paste as written. `~/.local/bin` is usually on the `PATH`; check with
`echo $PATH` if the command afterwards is not found:

```
mkdir -p ~/.local/bin
ln -s "$PWD/screendump" ~/.local/bin/screendump
screendump --list-midi
```

**Or call the file directly**, from the folder that holds it, and leave your
`PATH` alone:

```
./screendump --list-midi
python3 ./screendump --list-midi
```

Both work: the file is executable and starts with `#!/usr/bin/env python3`. You
need Python 3, which Linux ships as `python3`.

⚠️ **Snapshot and Record write into the folder you started from**, not next to
the script. Start it somewhere you will find the files again.

### What every command needs

`--instrument` is required and has no default: the machine decides what goes on
the wire, so the tool will not guess. Leave it out and it stops with *"no
instrument selected"* — which is the usual reason a first run appears to do
nothing at all.

**The window** needs only that one, because it finds its own ports:

```
screendump -x --instrument dtii --gui
```

The terminal modes need the two ports as well, either named:

```
screendump -x --instrument dtii --midi-in dtii --midi-out dtii --blocks
```

or set all three once, after which the shorter examples further down work as
written:

```
export SCREENDUMP_INSTRUMENT=dtii     # DT, DTII, DN -- see Instruments
export SCREENDUMP_MIDI_IN=dtii        # or an index from --list-midi
export SCREENDUMP_MIDI_OUT=dtii
```

The instrument can be written any accepted way, and a port takes an index, an
instrument short form, or part of a port name. [Set it once per
session](#set-it-once-per-session) has the detail, including other shells.

## Install

```
python3 -m pip install python-rtmidi
```

`python3 -m pip` and not bare `pip`, so the package lands in the interpreter
that runs the script. A second Python on the `PATH` is the usual reason for
"python-rtmidi is not installed" from a machine that has just installed it.

That one package is all you need to see a screen. The rest are needed only for
sound, video, or the window, and each is named when it is missing, so nothing
has to be installed up front:

| for | install |
|---|---|
| the screen, recording, GIF | `python3 -m pip install python-rtmidi` |
| `--save-audio` | `python3 -m pip install sounddevice` |
| `--save-video` | ffmpeg — `brew install ffmpeg`, `apt install ffmpeg`, `winget install ffmpeg` |
| `--gui` | tkinter — `sudo apt install python3-tk` (Debian/Ubuntu), `sudo dnf install python3-tkinter` (Fedora); part of Python elsewhere. See [A window instead of the terminal](#a-window-instead-of-the-terminal) |

## Find your MIDI ports

```
$ screendump --list-midi
MIDI inputs (instrument -> computer):
  [0] Elektron Digitakt
MIDI outputs (computer -> instrument):
  [0] Elektron Digitakt
```

## Capture the screen on an attached instrument

```
$ screendump -x --midi-in 0 --midi-out 0 --instrument dt
```

`-x` is the only mode that talks to hardware.

Those `0`s are indices from `--list-midi`, and on a machine with other MIDI gear
the instrument will not be number 0. Either read the number off that listing, or
pass the instrument's own name instead — `--midi-in dtii --midi-out dtii` — and
skip the lookup entirely.

Ports can be given three ways: an index from `--list-midi`, an instrument short
form (`--midi-in dt`), or any fragment of the port name (`--midi-in MOTU`, handy
when the instrument is behind a MIDI interface). A fragment has to identify
exactly one port — if two match, it stops and shows you both rather than
choosing for you.

`--instrument` takes only a machine a real screen has been captured from and
read: `dt` for a Digitakt, `dtii` for a Digitakt II. The others are known by
name and cannot be attempted — without an instrument id there is no request to
send. See [Instruments](#instruments).

Output goes to stdout, so it pipes and tees like anything else:

```
$ screendump -x --instrument dtii --midi-in dtii --midi-out dtii --blocks | tee shot.txt
```

That keeps one fixed rendering, as characters. To keep the **data** instead,
save the frame and re-open it — the pixels are still there, so you can render
them differently afterwards without going back to the instrument:

```
$ screendump -x --instrument dtii --midi-in dtii --midi-out dtii --save-syx shot.syx
$ screendump --instrument dtii --from-file shot.syx --blocks
$ screendump --instrument dtii --from-file shot.syx --ascii --invert
```

## Live view and recording

```
$ screendump -x --instrument dtii --midi-in dtii --midi-out dtii --framerate 30
```

Streams the screen, redrawing in place, until **Ctrl+C**. Your terminal is left
exactly as it was — the live view never enters the scrollback.

Every command from here on needs the same three things the capture above did:
`-x`, `--instrument`, and the two ports. See [Set it once per
session](#set-it-once-per-session) to put the ports in the environment and stop
repeating them.

For a window instead of the terminal — the way to get this screen into OBS —
see [A window instead of the terminal](#a-window-instead-of-the-terminal).

The rate is capped at 30, the instrument's own refresh, so asking for more just
returns the same frame twice. Over DIN expect about 3.

Over USB, `--bench` measures a clean 30 fps, while the live view settles nearer
26–27 once it is also drawing and recording. That costs you nothing in accuracy:
every frame is stamped with the moment it arrived, so a recording keeps its real
pacing whether the loop hit 30 or 26.

Since the screen refreshes at roughly 30 Hz, rates that divide into it — 30, 15,
10, about 7 — *may* sit more evenly against it than one in between, and a rate
like 25 can end up slower than either neighbour. Treat that as a rule of thumb
rather than exact science: if a recording looks uneven, try the nearest division
of 30.

### Record while you watch

```
$ screendump -x --instrument dtii --midi-in dtii --midi-out dtii \
             --framerate 15 --save-mid cap.mid --save-gif out.gif
                                    ... live view here, Ctrl+C to stop ...
screendump: 255 polled, 101 kept, 18.1 s
screendump: wrote 101 frames to cap.mid
screendump: wrote 101 frames (18.1 s) to out.gif
```

Only frames that actually **changed** are kept, each with the length of time it
was on screen — so a still screen costs one frame rather than fifteen a second.
That is the `255 polled, 101 kept` above.

Add `-H` / `--hide` to record without drawing anything.

### Or make the GIF afterwards

```
$ screendump --instrument dtii --from-file cap.mid --save-gif out.gif
```

A `.mid` keeps every frame and its timing, so you can come back to a recording
and re-cut it without touching the instrument again.

GIFs and videos are both enlarged four times by default — a 128 × 64 screen
becomes 512 × 256, since the raw size is nearly invisible on a modern display.
`--upscale N` changes it. Enlarging is nearest-neighbour on purpose: a 1-bit
screen through a smoothing filter turns to mush. It costs little in a GIF, where
4× is about four and a half times the bytes rather than sixteen.

⚠️ Some GIF viewers clamp very short delays, so a fast animation can play back
slower than it was captured.

## A window instead of the terminal

A terminal live view is for you. A window is for a video recorder: OBS can
capture it, and the screen is drawn at whatever size the window is.

```
$ screendump -x --instrument dtii --gui
```

The window needs **tkinter**, which is part of Python but is packaged
separately on Linux:

```
sudo apt install python3-tk        # Debian/Ubuntu
sudo dnf install python3-tkinter   # Fedora
sudo pacman -S tk                  # Arch
```

Nothing else is needed, and every other mode works without it.

It connects as soon as the ports resolve — the port carrying the instrument's
own name is picked for you, and both can be changed in the bar. If nothing is
picked, press **Connect**.

| | |
|---|---|
| status line | fps, frames polled, frames kept, and how long this has run. `NO REPLY` means the picture has stopped arriving, and what you see is the last frame |
| **Snapshot** | writes a PNG of the current screen, at the size it is drawn |
| **Record** | writes a `.mid` and a `.gif` into the current folder, named by the time it stopped |
| resize | whole pixels only — the window opens at 4× and grows in integer steps, so the screen never blurs |
| invert, flip, on top | swap light and dark, flip vertically, keep the window above other applications |

Keys: `s` snapshot, `r` record, `i` invert, `f` flip, `F11` fullscreen, `Esc`
leave fullscreen, `q` quit. In a replay, `space` pauses.

### A window with no instrument

Point it at a recording and it plays back at the speed it was captured — which
is also the way to try the window on a machine with no MIDI ports at all.

```
$ screendump --instrument dtii --from-file cap.mid --gui
```

⚠️ The window records **picture only**. Record writes a `.mid` and a `.gif`;
for sound, use the terminal: `screendump -x --framerate 30 --audio-in dt
--save-mid clip.mid --save-audio take.wav`. The window can then replay the
`.mid` while the pair is turned into a video in the usual way.

Tested on Linux with Tk 9.0. macOS and Windows have not been tried yet.

## Sound and video

A GIF shows what the instrument is doing. For what it *sounds* like you need
video, which means recording the audio at the same time.

This part needs two things the screen does not — recording audio needs

```
pip install sounddevice
```

and writing the video needs **ffmpeg** on your `PATH` (`brew install ffmpeg`,
`apt install ffmpeg`, `winget install ffmpeg`). Either one is named by the tool
if it is missing, so you can also just try it and see.

Find the audio input the same way as the MIDI ports:

```
$ screendump --list-audio
Audio inputs (instrument -> computer):
  [0] MacBook Air Microphone (Core Audio)   1 ch   48000 Hz
  [1] Elektron Digitakt (Core Audio)   2 ch   48000 Hz
```

`--audio-in` takes an index, an instrument short form, or any fragment of the
name — exactly like `--midi-in`. The host API is part of the name because the
same interface is listed once per host API on Windows, so `--audio-in
"Digitakt (WASAPI)"` is a way to pick one when several match.

**Record picture and sound together:**

```
$ screendump -x --instrument dtii --midi-in dtii --midi-out dtii \
             --framerate 30 --audio-in dt --save-mid cap.mid --save-audio cap.wav
```

**Then turn the pair into a video:**

```
$ screendump --instrument dtii --from-file cap.mid --from-audio cap.wav --save-video out.mp4
```

Video takes **two** inputs, because the two halves are recorded separately: a
`.mid` holds the picture and its timing, a `.wav` holds the sound. They line up
without you doing anything — the capture's clock starts at the first audio
sample, so both files share an origin.

Or do it in one go, and skip the intermediate files:

```
$ screendump -x --instrument dtii --midi-in dtii --midi-out dtii \
             --framerate 30 --audio-in dt --save-video out.mp4
```

Encoding is the slow part of a long recording — two minutes at the default size
is around half a gigabyte of raw frames handed to ffmpeg. `--upscale 2` quarters
that when you want a quick look rather than something to keep.

⚠️ **The first recording will ask for microphone permission** on macOS — that is
the OS gate on *any* audio input, including an instrument. If you refuse it, the
recording silently comes out empty.

### If sound and picture drift apart

Every recording prints what the audio clock actually did:

```
screendump: audio: 5001216 samples, 104.2 s, 0 overflows
screendump: audio clock: 47999.3 Hz measured vs 48000 Hz nominal (-15 ppm, -2 ms over this capture)
```

An interface that says 48000 Hz is really a crystal a few tens of parts per
million away from that, while the frame times come from the computer's clock.
The two disagree slowly, so the figure is measured and printed rather than
assumed. That last number is how far the sound has slid against the picture.

The figures above are a real Digitakt over USB: **−15 ppm, about a millisecond a
minute**, so it would take roughly an hour to accumulate 50 ms. In practice this
does not matter — but it is measured rather than promised.

`0 overflows` is the other number to glance at. Anything above zero means
samples were **dropped**, and the recording is short by that much. A Digitakt
polled at 30 fps for 104 seconds dropped none, so screenshotting the instrument
does not disturb its audio.

If you pair a `.wav` with a `.mid` from a different capture, you get told:

```
screendump: ⚠️ cap.wav is 16.9 s but the recording is 104.2 s -- are they from the same capture?
```

The video always follows the **picture**; sound that runs short is padded with
silence and sound that runs long is trimmed.

## Set it once per session

Each variable takes exactly what its option takes — so the `0`s below are port
**indices** from `--list-midi`, not "off" or "default", and the instrument can
be written any of its three accepted ways.

```
$ export SCREENDUMP_MIDI_IN=0             # [0] from --list-midi
$ export SCREENDUMP_MIDI_OUT=0            # [0] from --list-midi
$ export SCREENDUMP_AUDIO_IN=dt           # short form, or an index from --list-audio
$ export SCREENDUMP_INSTRUMENT=Digitakt   # or DT, or dt -- all three work

$ screendump -x
```

An option always beats the environment. Setting variables differs by shell:

| shell | |
|---|---|
| sh / bash / zsh | `export SCREENDUMP_MIDI_IN=0` |
| PowerShell | `$env:SCREENDUMP_MIDI_IN = "0"` |
| cmd.exe | `set SCREENDUMP_MIDI_IN=0` |

## Options

| | |
|---|---|
| `-x`, `--external` | capture from the instrument |
| `--list-midi` | list MIDI ports |
| `--list-audio` | list audio inputs |
| `--midi-in`, `--midi-out` | port index or name fragment |
| `--audio-in` | audio input index or name fragment |
| `--instrument` | which machine — required, no default |
| `--blocks` | half-block rendering, no row gaps |
| `--ascii` | full resolution, one character per pixel |
| `--invert`, `--flip` | swap lit/unlit, flip vertically |
| `--gui` | show the screen in a window instead of the terminal: live with `-x`, or a replay with `--from-file` |
| `--framerate N` | with `-x`, stream until Ctrl+C (capped at 30); also the frame rate of a video |
| `-H`, `--hide` | while streaming, don't draw to the terminal |
| `--save-syx PATH` | also save the frame |
| `--save-raw PATH` | also save just the pixel bytes |
| `--save-mid PATH` | also save the recording, with timing, as a MIDI file |
| `--save-gif PATH` | also save an animated GIF |
| `--save-audio PATH` | also record the instrument's audio to a stereo WAV |
| `--save-video PATH` | write an mp4 of picture and sound together |
| `--upscale N` | enlarge each pixel to N × N in a GIF or video, and the window's opening size (default 4) |
| `--from-file PATH` | render a saved `.syx`, `.mid` or raw dump instead |
| `--from-audio PATH` | the `.wav` to pair with `--from-file` for a video |
| `--bench N` | measure the frame rate over N captures |
| `--self-test` | check the tool without an instrument |
| `--timeout` | reply timeout in seconds, default 2 |

Nothing is written to disk unless you ask.

## If braille looks gappy

The default rendering uses braille, which is compact but shows seams between
rows in many terminals, because braille dots sit inset in the character cell.

**Use `--blocks`.** It fills the cell edge to edge and always looks solid.

To keep braille instead, reduce your terminal's line spacing:

| terminal | setting |
|---|---|
| iTerm2 | Profiles → Text → Vertical Spacing → 100 % or below |
| Ghostty | `adjust-cell-height = -8%` |
| kitty | `modify_font cell_height -2px` |
| WezTerm | `line_height = 1.0` |
| Terminal.app | not adjustable — use `--blocks` |

Iosevka and JuliaMono draw tight braille; SF Mono and Menlo are heavily padded.

## Instruments

`--instrument` is spelled exactly — the short form, its lowercase form, or the
full name. `digitakt` and `DigiTAKT` are rejected on purpose: the instrument
decides what goes on the wire, so a typo should be an error rather than a guess.

**Implemented**

| | | |
|---|---|---|
| `DT` | `dt` | Digitakt |
| `DTII` | `dtii` | `"Digitakt II"` |

The Digitakt II answers the Digitakt request and reports the same 128 × 64
screen, so one capture path serves both and either name works.

**Known, not implemented**

| | | |
|---|---|---|
| `DN` | `dn` | `Digitone` |
| `DNII` | `dnii` | `"Digitone II"` |
| `ST` | `st` | `Syntakt` |
| `AR` | `ar` | `"Analog Rytm"` |
| `A4` | `a4` | `"Analog Four"` |
| `AH` | `ah` | `"Analog Heat"` |
| `M:S` | `m:s` | `Model:Samples` |
| `M:C` | `m:c` | `Model:Cycles` |
| `OT` | `ot` | `Octatrack` |
| `MD` | `md` | `Machinedrum` |

Asking for one of these stops with *"The Octatrack is not implemented for
screendump yet"*. That is not caution — no instrument id is known for it, so
there is no request to send.

Names containing a space need quoting: `--instrument "Digitakt II"`. The short
forms never do, which is the easier habit.

An instrument moves to the implemented list once a screen has been captured from
it and read. Adding one is a few lines — see the notes at the top of the script.
