# Sebby's Music Box — project handoff

Last verified: 2026-07-20

## What this is

Sebby's Music Box is a self-contained browser instrument in `index.html`. It combines a hand-controlled chord wheel, polyphonic Web Audio synth, arpeggiator, step drum machine, MusicXML chord chart, audio-reactive visuals, modulation routing, presets, and recording. It has no build step.

Run it from a local web server (camera and ES modules are unreliable from a raw `file://` URL), then open the page in a modern Chromium-based browser.

## What is finished

- Chord parser for named and roman-numeral harmony, including extensions, alterations, inversions, and up to 16 wheel slots.
- Voice-led three-oscillator pad synth with editable harmonics, ADSR, glide, vibrato, filter, chorus, reverb, and delay.
- Two-hand MediaPipe tracking. Right-hand position selects the wheel; default mappings are right pinch → volume and right tilt → vibrato.
- Universal modulation popup: double-click any `⌁` label to map it to either hand or one of two LFOs. ADSR stages and mixer controls are included.
- Arpeggiator with its own harmonic editor, decay, octave, modes, free rate, tempo divisions, and shared swing/transport timing.
- Four-voice, 16-step drum sequencer with preset patterns, mute controls, tempo, swing, keyboard transport, and look-ahead scheduling.
- Per-instrument mixer strips for chord pad, arp, and drums: level, pan, tone, reverb send, and delay send.
- Lead-sheet-style MusicXML chart. Upload a file, click chart chords to audition, and load unique song harmony into the wheel.
- Five audio-reactive canvas visualizers with camera blending.
- Factory presets (`Aqua Bloom`, `Glass Arp`, `Moon Pond`, `House Halo`) plus local save/load, JSON import/export, and complete mixer/modulation state.
- Audio recording through `MediaRecorder`.
- Camera-free mode with number-key and mouse interaction.
- Light/dark themes, Y2K/aqua-glass intro, and a seven-step animated tutorial available from the intro or the `?` button.

## Important implementation notes

- `state` holds patch-level values. Runtime-only audio objects live in the audio-engine section.
- `state.mixer` is the source of truth for each instrument strip. `setStrip()` applies values to live Web Audio nodes.
- Modulation never overwrites base values. `applyMods()` recomputes effective values every animation frame and calls registered apply functions.
- Drum and synced-arp events share `transportOrigin`; do not give the arp a separate timer or swing calculation.
- The visible overlay canvas uses device-pixel coordinates. Hand points and mouse points are converted to that same coordinate space before wheel hit-testing.
- Preset snapshots must remain plain JSON. Convert harmonic arrays back to `Float32Array` when applying a snapshot.
- Keep all optional-control wiring null-safe. A removed panel control previously stopped the entire ES module before boot, which made the wheel appear to be gone.

## Verification checklist

1. Start with **no camera** and confirm the seven-slice wheel appears.
2. Click and hold a slice; confirm chord and volume readouts change and audio sounds.
3. Open Settings; verify oscillator, ADSR, LFO, mixer, arp, drum, visual, chord, and preset controls.
4. Double-click a `⌁` mixer or envelope label; map it to an LFO and confirm the value badge animates.
5. Load each factory preset and confirm theme/visual, mixer, arp, and drum state update without console errors.
6. Run the guided tour through all seven steps.
7. Start the drums and a synced arp; verify swing affects their offbeats without tempo drift.
8. If camera testing is available, start with camera permission and verify one- and two-hand behavior.

## Known limitations / sensible next work

These are enhancements, not missing parts of the current request:

- The MusicXML reader uses the first part and harmony tags; it is not a full notation renderer.
- Keyboard shortcuts directly address the first ten wheel slices. Slices 11–16 remain available by hand/mouse.
- Presets are stored in browser `localStorage`; there is no cloud sync.
- Mobile layout works best in landscape, but the dense settings panel could use a dedicated narrow-screen pass.
- Automated audio assertions are limited by browser autoplay and camera permission policies, so final sound/camera QA still benefits from a human pass.
