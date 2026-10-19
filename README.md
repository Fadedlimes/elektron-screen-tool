# screendump / Elektron-Screen-Tool

Display and record your Elektron instrument's screen in real time directly from your terminal or in a dedicated desktop window. Designed for screen captures, streaming, and video production in OBS without glare, ghosting, or duplicate video capture devices.

<img width="512" height="256" alt="screendump-20261018-223646" src="https://github.com/user-attachments/assets/80953731-1989-4879-9edf-70e444fa57ab" />

`screendump` queries the device's native screenshot routine over MIDI SysEx. **No firmware modifications, patching, or flashing required.**

- **Platforms:** macOS and Linux supported (Windows is unverified but should work).
- **Supported Instruments:** Digitakt (I & II). Additional Elektron devices can be probed and mapped via `--trace`.

> **Credit:** Core MIDI capture routines, protocol decoding, and export engines are based on [Jakob Bak's screendump](https://github.com/jakobbak/screendump) (MIT License). This fork adds Digitakt II support, an interactive GUI window suitable for OBS window capture, and synchronized audio recording.

---

### GUI Preview

<img width="641" height="467" alt="image" src="https://github.com/user-attachments/assets/aa9c0aac-a86a-4fc6-ac49-f2f3c1de6565" />



---

## Installation & Dependencies

`screendump` is distributed as a single standalone Python 3 script.

### 1. Python Dependencies

Install the core MIDI library:

```bash
python3 -m pip install python-rtmidi
```

*(Note: Always use `python3 -m pip` to ensure the library is installed into the matching Python interpreter environment.)*

#### Optional Dependencies

Install additional tools only if you need audio recording, video rendering, or the graphical window:

| Feature | Dependency | Installation Command |
|---|---|---|
| Core Screen Capture, `.mid`, `.gif` | `python-rtmidi` | `python3 -m pip install python-rtmidi` |
| Audio Capture (`--save-audio`) | `sounddevice` | `python3 -m pip install sounddevice` |
| Video Export (`--save-video`) | `ffmpeg` | `brew install ffmpeg` / `sudo apt install ffmpeg` / `winget install ffmpeg` |
| Graphical UI (`--gui`) | `tkinter` | `sudo apt install python3-tk` (Ubuntu/Debian)<br>`sudo dnf install python3-tkinter` (Fedora)<br>*Included by default with official macOS/Windows Python installers.* |

---

## Running the Tool

You can run the script directly from your terminal or link it into your system `$PATH`:

### Option A: Run directly
```bash
python3 ./screendump --list-midi
# or make it executable:
chmod +x ./screendump
./screendump --list-midi
```

### Option B: Add to your PATH
To run `screendump` from any directory:
```bash
mkdir -p ~/.local/bin
ln -s "$PWD/screendump" ~/.local/bin/screendump
```
*(Ensure `~/.local/bin` is present in your shell's `$PATH` variable).*

> ⚠️ **Note on Output Files:** Screenshots, recordings, and audio files are saved to your **current working directory** (wherever your terminal prompt is currently located), not in the directory where `screendump` is stored.

---

## Basic Usage

### 1. Identify Your MIDI Ports

List available MIDI input and output ports:

```bash
$ screendump --list-midi
MIDI inputs (instrument -> computer):
  [0] Elektron Digitakt
MIDI outputs (computer -> instrument):
  [0] Elektron Digitakt
```

### 2. Capture a Single Screenshot

Use `-x` (or `--external`) to interact with connected hardware:

```bash
screendump -x --instrument dtii --midi-in 0 --midi-out 0
```

- **Specifying Ports:** You can pass the numerical index (e.g., `0`), an instrument short code (`dtii`), or any unique substring of the port name (e.g., `--midi-in "Digitakt"`). If an ambiguous name matches multiple ports, the tool lists matches and exits.
- **Specifying Instruments:** `--instrument` is required when capturing from hardware. See [Supported Instruments](#supported-instruments).

#### Exporting & Formats:
Output defaults to standard output (`stdout`), allowing pipes and redirection:

```bash
# Render to terminal using Unicode half-blocks:
screendump -x --instrument dtii --midi-in dtii --midi-out dtii --blocks

# Save the raw SysEx response to replay or re-render later:
screendump -x --instrument dtii --midi-in dtii --midi-out dtii --save-syx shot.syx

# Render from an existing SysEx file without hardware connected:
screendump --instrument dtii --from-file shot.syx --blocks
screendump --instrument dtii --from-file shot.syx --ascii --invert
```

---

## Live Streaming & Recording

Stream the device display live to your terminal at up to 30 FPS:

```bash
screendump -x --instrument dtii --midi-in dtii --midi-out dtii --framerate 30
```
*(Press `Ctrl+C` to stop. Terminal scrollback is preserved.)*

- **USB vs. 5-pin DIN:** Over USB MIDI, capture speed reaches 26–30 FPS. Over standard 5-pin DIN MIDI, bandwidth limitations cap transfer speeds at roughly 3 FPS.
- **Frame Pacing:** Because the native hardware refresh rate is ~30 Hz, evenly divisible target rates (`30`, `15`, `10`) yield smoother playback than non-integer divisors (like `25`).

### Record Animated GIFs

```bash
screendump -x --instrument dtii --midi-in dtii --midi-out dtii \
  --framerate 15 --save-mid session.mid --save-gif session.gif
```

- `screendump` only records frames that change, storing timestamps for accurate variable-framerate playback. A static screen produces no redundant duplicate frames.
- Add `-H` / `--hide` to disable terminal rendering while recording.
- By default, GIFs are upscaled 4× (`--upscale 4`) using nearest-neighbor scaling to keep the 1-bit display sharp and prevent blur.

---

## Graphical Window Mode (`--gui`)

For OBS capture or clean desktop viewing, launch the Tkinter graphical window:

```bash
screendump -x --instrument dtii --gui
```

When only one Elektron device is connected via USB, the GUI automatically identifies and pairs the matching MIDI in/out ports.

### GUI Controls & Features
- **Window Scaling:** Resizes in integer pixel multipliers (4×, 5×, etc.) to prevent scaling artifacts and distortion.
- **Status Line:** Displays active FPS, polled frames, and status warnings. `NO REPLY` indicates communication with the instrument was interrupted.
- **Snapshot (`s`):** Saves a PNG screenshot at current display resolution.
- **Record (`r`):** Records incoming frames to a `.mid` and `.gif` file in your current folder.
- **Hotkeys:** 
  - `i`: Invert colors
  - `f`: Flip vertically
  - `F11`: Toggle fullscreen
  - `Space`: Pause/resume playback (when viewing a file)
  - `q`: Quit

> ⚠️ **Port Conflicts with Elektron Transfer:** If the official **Elektron Transfer** desktop utility is open, it holds an exclusive lock on the USB-MIDI ports. Quit Elektron Transfer and click the refresh button (`⟳`) in `screendump` to regain access.

### Offline GUI Playback
You can inspect or replay existing captures without an instrument attached:

```bash
screendump --instrument dtii --from-file session.mid --gui
```

---

## Synchronized Audio & Video Recording

Record display captures combined with direct USB audio from your Elektron instrument:

### 1. List Available Audio Inputs
```bash
$ screendump --list-audio
Audio inputs (instrument -> computer):
  [0] Built-in Microphone (Core Audio)   1 ch   48000 Hz
  [1] Elektron Digitakt (Core Audio)     2 ch   48000 Hz
```

### 2. Record Video and Audio Directly
```bash
screendump -x --instrument dtii --midi-in dtii --midi-out dtii \
  --framerate 30 --audio-in "Digitakt" --save-video output.mp4
```

### 3. Or Record Components Separately and Merge:
```bash
# Capture raw MIDI frames and audio to disk:
screendump -x --instrument dtii --midi-in dtii --midi-out dtii \
  --framerate 30 --audio-in "Digitakt" --save-mid session.mid --save-audio session.wav

# Render the files to an MP4 video using ffmpeg:
screendump --instrument dtii --from-file session.mid --from-audio session.wav --save-video output.mp4
```

> **macOS Audio Permissions:** On macOS, the terminal or application running Python must be granted **Microphone Access** in *System Settings → Privacy & Security*. Without this, audio captures will be silent.

### Audio Clock Sync & Drift Correction
Hardware audio clocks (typically 48,000 Hz) run on independent quartz crystals that drift slightly relative to your computer's system clock. `screendump` records the exact audio clock drift upon completion:

```text
screendump: audio: 5001216 samples, 104.2 s, 0 overflows
screendump: audio clock: 47999.3 Hz measured vs 48000 Hz nominal (-15 ppm, -2 ms over this capture)
```
A drift of ~15 ppm equals ~1 millisecond per minute, which is imperceptible in practice. If the audio input encounters buffer overflows, a warning will report dropped samples.

---

## Environment Variables (Persistent Defaults)

To avoid specifying ports and instrument flags on every command, set shell environment variables:

```bash
# Bash / Zsh:
export SCREENDUMP_INSTRUMENT=dtii
export SCREENDUMP_MIDI_IN=dtii
export SCREENDUMP_MIDI_OUT=dtii
export SCREENDUMP_AUDIO_IN=dtii

# Run without repetitive flags:
screendump -x
```

#### Syntax by Shell:
| Shell | Command |
|---|---|
| **sh / bash / zsh** | `export SCREENDUMP_INSTRUMENT="dtii"` |
| **PowerShell** | `$env:SCREENDUMP_INSTRUMENT = "dtii"` |
| **cmd.exe** | `set SCREENDUMP_INSTRUMENT=dtii` |

*(Explicit command-line arguments always override environment variables.)*

---

## Command Reference

| Flag | Description |
|---|---|
| `-x`, `--external` | Enable communication with connected hardware. |
| `--list-midi` | List detected MIDI input and output ports. |
| `--list-audio` | List detected system audio capture devices. |
| `--midi-in`, `--midi-out` | Select MIDI port by index, shortcode, or name fragment. |
| `--audio-in` | Select audio input by index, shortcode, or name fragment. |
| `--instrument` | Target instrument identifier (required for live capture). |
| `--gui` | Launch the graphical display window instead of terminal rendering. |
| `--framerate N` | Set target polling framerate for live capture (max 30). |
| `-H`, `--hide` | Suppress terminal drawing during capture. |
| `--blocks` | Render terminal output using half-block Unicode characters (`▀`, `▄`). |
| `--ascii` | Render terminal output using 1:1 ASCII characters. |
| `--invert`, `--flip` | Invert display colors or flip image vertically. |
| `--upscale N` | Upscale factor for GIF/video exports and initial GUI size (default: `4`). |
| `--save-syx PATH` | Save received raw SysEx frame to disk. |
| `--save-raw PATH` | Save decoded pixel buffer to disk. |
| `--save-mid PATH` | Save timed capture stream as a standard MIDI file. |
| `--save-gif PATH` | Save capture session as an animated GIF. |
| `--save-audio PATH` | Save captured audio stream to a 24-bit stereo `.wav` file. |
| `--save-video PATH` | Encode capture session to an `.mp4` video (requires `ffmpeg`). |
| `--from-file PATH` | Replay or re-render an existing `.syx`, `.mid`, or raw capture. |
| `--from-audio PATH` | Audio `.wav` track to pair with `--from-file` for video export. |
| `--trace` | Send discovery SysEx packets to unknown instruments. |
| `--probe-id "XX YY"` | Two-byte SysEx machine ID to test with `--trace` (default: `10 00`). |
| `--bench N` | Measure throughput and framerate across `N` capture frames. |
| `--timeout SEC` | Response timeout in seconds (default: `2`). |

---

## Terminal Rendering Tips

Default terminal rendering uses Unicode Braille characters. On some terminals, font metrics create visible horizontal gaps between rows.

- **Recommended Fix:** Pass `--blocks` for an edge-to-edge block character view.
- **To fix Braille line height in your terminal emulator:**
  - **iTerm2:** *Profiles → Text → Vertical Spacing* set to `100%` or lower.
  - **Ghostty:** Add `adjust-cell-height = -8%` to your configuration.
  - **kitty:** Add `modify_font cell_height -2px` to `kitty.conf`.
  - **WezTerm:** Set `line_height = 1.0` in configuration.
  - **macOS Terminal.app:** Line spacing cannot be adjusted; use `--blocks`.

---

## Supported Instruments

| Model | Code | Alternate Aliases | Status |
|---|---|---|---|
| **Digitakt** | `dt` | `DT`, `"Digitakt"` | Supported |
| **Digitakt II** | `dtii` | `DTII`, `"Digitakt II"` | Supported |
| **Digitone** | `dn` | `DN`, `"Digitone"` | Unmapped |
| **Digitone II** | `dnii` | `DNII`, `"Digitone II"` | Unmapped |
| **Syntakt** | `st` | `ST`, `"Syntakt"` | Unmapped |
| **Analog Rytm** | `ar` | `AR`, `"Analog Rytm"` | Unmapped |
| **Analog Four** | `a4` | `A4`, `"Analog Four"` | Unmapped |
| **Analog Heat** | `ah` | `AH`, `"Analog Heat"` | Unmapped |
| **Model:Samples** | `m:s` | `M:S`, `"Model:Samples"` | Unmapped |
| **Model:Cycles** | `m:c` | `M:C`, `"Model:Cycles"` | Unmapped |
| **Octatrack** | `ot` | `OT`, `"Octatrack"` | Unmapped |
| **Machinedrum** | `md` | `MD`, `"Machinedrum"` | Unmapped |

---

## Adding Support for New Instruments (`--trace`)

Elektron uses proprietary two-byte SysEx identification headers for each hardware model. These IDs are not published in user manuals or standard firmware updates.

If you own an unsupported device, you can safely probe for its screenshot ID using `--trace`:

```bash
# Probes the connected device using a specific candidate ID:
screendump --trace --midi-in 1 --midi-out 1 --probe-id "0c 00"
```

1. `--trace` sends a single read request over MIDI. It will **not** write to or modify your device.
2. If the device recognizes the probe, it responds with display dimensions and pixel payloads.
3. The tool generates two diagnostic dump files:
   ```text
   screendump-trace-YYYYMMDD-HHMMSS.txt
   screendump-trace-YYYYMMDD-HHMMSS.syx
   ```
4. If the preview image printed in your terminal matches what is on your screen, submit these two files via a [New Device Issue](https://github.com/Fadedlimes/elektron-screen-tool/issues/new/choose) to have your model added to the supported table.
