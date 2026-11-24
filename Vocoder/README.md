# Wavetable Vocoder

A single-file browser wavetable synth in the spirit of the Fairlight CMI and PPG Wave. A flute note is stored as 32 single-cycle waveforms, and the synth steps through them as the note plays. A 3D waterfall display shows the whole stack of waves, with the current position highlighted as it moves.

It also doubles as the carrier for a channel vocoder: speak or play a voice recording into it and the wavetable sound appears to talk.

No build step, no dependencies. Open `wavetable-vocoder.html` in a modern browser.

## Quick start

1. Open `wavetable-vocoder.html` (double-click it, or serve the folder with any static server).
2. Click the page once so it has keyboard focus.
3. Press **A** to play a note. The status line above the stack should read "Audio running".

## Playing

| Control | What it does |
| --- | --- |
| **A S D F G H J K** | White keys, one octave from C |
| **W E T Y U** | Black keys |
| **Z / X** | Octave down / up (range of -2 to +2) |
| On-screen keys | Same notes, for touch and mouse |

The synth is polyphonic, so you can hold several keys at once.

## Controls

- **Sweep / Hold**
  - *Sweep* plays through the table from the first frame to the last over the sweep time, then holds on the last frame.
  - *Hold* locks playback to the scan position slider (or whichever frame you click in the stack).
- **Sweep time:** how long a note takes to travel from first frame to last (0.3 to 8 s).
- **Scan position:** the frame being played, 0 to 100%. Moves on its own in Sweep mode and is draggable in Hold mode.
- **Sustain loop (Sweep mode only):** once the sweep reaches the "to" point, the scan bounces back and forth between "from" and "to" instead of stopping, so a decaying sound like a pluck keeps going while you hold the key. **Loop speed** sets how fast it bounces.
- **Release:** how long notes take to fade after you let go (0.05 to 2 s).
- **Waveforms / Harmonics:** switches the 3D stack between the raw waveforms and a harmonic-amplitude waterfall for each frame.

## Importing a recording

Click **Load audio…** (or drop a file on the 3D stack) to replace the built-in table with your own sound.

- **Formats:** WAV is the safe choice. MP3, OGG and M4A/AAC work wherever the browser can decode them.
- **Best source:** one sustained, monophonic note with a fairly steady pitch, 1 to 8 seconds long. Chords, heavy reverb and big pitch glides will smear.
- **What happens:** the file is mixed to mono, trimmed to the audible part, its pitch is detected, and 32 single cycles are cut along the note. Each is resampled to 512 samples and phase-aligned to its neighbour. The slice stops where the sound falls to about 6% of its peak, and the sweep time is set to that length.
- **Attack detail:** spaces the 32 frames unevenly, packing more of them into the start of the note where a pluck or chiff changes fastest. The sweep is timed to match, so the note still plays at its real speed. It is set automatically (0% for steady notes, 50% for decaying ones) and re-slices the recording when you move it. Re-slicing discards manual frame edits.
- **Level each frame:** on, every frame is normalised for an even sound. Off, the original loudness shape of the recording is kept.
- **Pitch:** the A key plays the recording at its original pitch (the detected note, with its small tuning offset applied), and the other keys follow chromatically. Z / X shift octaves. The detected pitch is shown next to the button. If it is an octave out, the detection picked the wrong period.
- **Reset table** brings back the built-in flute.

## Vocoder

Scroll below the editor to the vocoder. The voice is split into 8, 16 or 24 log-spaced bands (120 Hz to 7.5 kHz). Each band has an envelope follower, and its level opens the matching band of the carrier, which is what makes the synth sound like it is speaking. The live band levels are drawn on the canvas.

**To use it:**

1. Optionally load your own recording into the wavetable section first.
2. Click **Use microphone** (wear headphones to avoid feedback) or **Load voice file…** to loop a recording as the voice.
3. Hold keys A to K, or switch on **Drone chord** for hands-free use, and talk.

The original voice is never played, only the vocoded result.

**Voice controls**

- **Input gain:** level going into the bands.
- **Bands:** 8 is robotic, 24 is more intelligible.
- **Smoothing:** how quickly the band envelopes follow the voice. Higher is snappier, lower is smoother.

**Carrier controls**

- **Source:** *Wavetable → vocoder* (default), *Plain saw → vocoder* (a detuned saw or square, for comparison), or *Wavetable dry* (the wavetable straight to the speakers, vocoder off).
- **Brightness:** a high-shelf boost on the wavetable carrier. Flutes and plucks are dark, so the upper bands need help to carry speech.
- **Formant shift:** moves the carrier bands up or down relative to the voice bands, from -12 to +12 semitones.
- **Noise in carrier:** mixes noise into the carrier so every band has something to shape.
- **Consonants:** highpass-filtered voice energy opens a burst of noise, which brings back S, T and F sounds.
- **Output:** overall level.

**Tips**

- Use a carrier that sustains. Turn on the sustain loop for plucks, or switch to Hold mode on a bright frame for the steadiest, clearest speech.
- Sweep mode restarts the timbre on every key press, so the sound evolves as you talk. It sounds great but is less intelligible.
- Speak clearly and close to the mic, and set Input gain so the bars move without constantly hitting the top.

## Reading the 3D stack

Each line is one frame of the table. The first frame (start of the note) is at the front, and later frames recede up and to the right, with nearer frames hiding the ones behind them. Colours:

- **Pink:** the frame currently being played
- **Teal:** the frame selected for editing
- **Grey-violet:** everything else, fading with depth

Click or drag on the stack to select a frame. In Hold mode this also sets the scan position.

## Editing

The strip below the stack shows the selected frame. Draw on it with the mouse or a finger to redraw the waveform. The stack and the sound update immediately.

- **Smooth frame** applies a light 3-point smoothing to the selected frame.
- **Reset table** regenerates the original flute table and discards your edits.

Edits live in memory only. Reloading the page resets everything.

## How it works

- **The table:** 32 frames of 512 samples each. It is generated by additive synthesis: 40 harmonics with a gentle rolloff, a brighter and breathier attack that decays over the first third of the note, and a slow shimmer in the upper harmonics. Attack phases are randomised and settle toward alignment, so the sustain frames crossfade smoothly while the attack stays textured.
- **Playback:** each voice reads the table at a rate set by pitch. The scan position picks two neighbouring frames, and the output crossfades between them, using linear interpolation both within each frame and between frames.
- **Audio engine:** a `ScriptProcessorNode` renders all voices on the main thread, with a short attack ramp, an exponential release and `tanh` soft clipping on the sum.
- **Display:** two `<canvas>` elements. The stack is drawn back to front, each frame filled with the panel colour before its line is stroked, which gives hidden-line removal. Colours follow your system light or dark theme.

## Known limitations

- **The built-in table is synthetic.** Import a recording to use real material. Slicing is automatic and works best on clean, steady notes.
- **No band-limiting.** High notes can alias because the table is read with plain linear interpolation and no per-octave versions.
- **`ScriptProcessorNode` is deprecated.** It was chosen because it works where blob-based AudioWorklets can be blocked. It runs on the main thread, so heavy page activity can cause glitches.
- **Vocoder fidelity.** Intelligibility is limited by band count, carrier brightness and mic quality. The envelope followers are symmetric, with no separate attack and release.
- **Microphone access** can be blocked inside embedded pages. Use **Load voice file…** in that case.
- **Performance.** The 3D display plus the filter bank is heavy on phones. Moving the wavetable engine to an AudioWorklet would help.
- **No persistence.** Edited tables are not saved or exportable yet.
- **Keyboard shortcuts need page focus.** Click the page first. On iPhones, the ringer switch mutes web audio.

## Ideas for next steps

- Band-limited mipmapped tables, or a move to an AudioWorklet
- Scan modulation (LFO, velocity, mod wheel) and a filter
- Save and load tables, and export frames as WAV
- Harmonic-bar editing, Fairlight style

## Files

- `wavetable-vocoder.html`: the merged app (HTML, CSS and JavaScript in one file)
- `wavetable-flute.html`: the wavetable synth on its own (or channel vocoder.html)
- `vocoder.html`: the vocoder on its own, with a plain saw carrier
- `README.md`: this file

## Browser support

Any current Chrome, Edge, Firefox or Safari. Uses Web Audio, Canvas 2D and Pointer Events. Fonts load from Google Fonts when online and fall back to the system sans-serif otherwise.