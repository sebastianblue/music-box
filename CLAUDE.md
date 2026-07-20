# Sebby's Music Box — project handoff

Last verified: 2026-07-20

## What this is

Sebby's Music Box is a self-contained browser instrument in `index.html`. It combines a hand-controlled chord wheel, polyphonic Web Audio synth, shared-transport chord looper, arpeggiator, step drum machine, MusicXML chord chart, audio-reactive visuals, modulation routing, presets, audio recording, and MIDI handoff. It has no build step.

Run it from a local web server (camera and ES modules are unreliable from a raw `file://` URL), then open the page in a modern Chromium-based browser.

## What is finished

- Chord parser for named and roman-numeral harmony, including extensions, alterations, inversions, and up to 16 wheel slots.
- Voice-led three-oscillator pad synth with editable harmonics, ADSR, glide, vibrato, filter, chorus, reverb, and delay.
- Two-hand MediaPipe tracking. Right-hand position selects the wheel; default mappings are right pinch → volume and right tilt → vibrato.
- Universal modulation popup: double-click any `⌁` label to map it to either hand or one of two LFOs. ADSR stages and mixer controls are included.
- Hand-mapped universal controls automatically appear as live top-left performance chips beside Volume and Vibrato. Each chip shows its target, hand source, formatted output value, and active/inactive tracking state; hand-controlled values and movement also influence all five visualizers.
- Performance module with tap tempo, audible one- or two-bar count-in, optional metronome, last-chord hold, four bass behaviors, octave register, voicing spread, and timing humanization. Register, spread, and humanize are universal hand/LFO mapping targets.
- Sixteen-beat chord looper with 4/8/16-beat lengths, live beat-quantized wheel recording, direct chord editing/auditioning, four progression starters, wheel-to-loop and loop-to-wheel transfer, rests, transport position, and a live top-left status chip. Loop playback drives the synth, voice-led MIDI, wheel/readout, chart highlights, arp, and visualizers.
- Arpeggiator with its own harmonic editor, decay, octave, modes, free rate, tempo divisions, and shared swing/transport timing.
- Eight-voice, 8/16/32-step drum sequencer with velocity accents, per-track mute/solo/level/tuning/decay/probability, six pattern tools, page follow/copy-paste, tempo, swing, keyboard transport, and look-ahead scheduling.
- Per-instrument mixer strips for chord pad, arp, and drums: level, pan, tone, reverb send, and delay send.
- Web MIDI output for recording the chord wheel or chord loop into Logic Pro through a macOS IAC bus. The MIDI module supports output selection, channels 1–16, octave transpose, velocity, expression (CC11), vibrato/mod-wheel (CC1), a test note, panic, and optional local-audio monitoring.
- Standard MIDI file export that works without a live MIDI connection. It writes a tempo/time-signature track, voiced chord track, and General MIDI drum track, repeating shorter patterns to the shared arrangement length and preserving swing, deterministic humanization, accents, track levels, mute, and solo state.
- Lead-sheet-style MusicXML chart. Upload a file, click chart chords to audition, and load unique song harmony into the wheel.
- Five audio-reactive canvas visualizers with camera blending.
- Fifteen factory worlds grouped as Ambient, Keys, Rhythmic, Club, Experimental, and Studio. Every world has distinct key-centered wheel harmony, chord-loop writing, voicing/bass behavior, tempo, sound, rhythm, mappings, mix, and visual treatment. The preset library includes descriptions/tags, previous/random/next browsing, and scoped recall for Everything, Sound & mix, Groove & chords, or Visual only.
- Personal preset save/load plus JSON import/export. Scoped sound/visual recall preserves an active chord-loop recording position and held chord instead of interrupting the performance.
- Audio recording through `MediaRecorder`.
- Camera-free mode with number-key and mouse interaction.
- Light/dark themes, Y2K/aqua-glass intro, and an eleven-step learn-by-doing tutorial available from the intro or the `?` button. It spotlights live controls, waits for each action, and enables the camera automatically when the hand-routing step needs it.
- Every settings module can detach into a draggable, resizable Frutiger Aero window with minimize, maximize, focus stacking, and one-click docking. The original live controls move between the panel and window, so state and event wiring are preserved.

## Important implementation notes

- `state` holds patch-level values. Runtime-only audio objects live in the audio-engine section.
- `state.mixer` is the source of truth for each instrument strip. `setStrip()` applies values to live Web Audio nodes.
- `state.performance` holds latch, metronome/count-in, register, spread, humanize, and bass-mode patch values. `registerEff`, `spreadEff`, and `humanizeEff` are recomputed by the modulation pass.
- Modulation never overwrites base values. `applyMods()` recomputes effective values every animation frame and calls registered apply functions.
- Drum and synced-arp events share `transportOrigin`; do not give the arp a separate timer or swing calculation.
- `chordLoop` stores sixteen plain chord-symbol strings or `null` rests. Its length is measured in quarter-note beats. `chordStepQueue` changes the live pad/MIDI chord at actual audio time, while `transportChordAt()` gives the look-ahead arp the correct future chord.
- Voicing register/spread history is tracked by `appliedRegisterShift` and `appliedSpreadBucket`; this prevents repeated chord changes from accumulating octave shifts.
- MIDI export is deliberately self-contained and dependency-free. Keep it Standard MIDI File format 1 with tempo, chord, and drum tracks; drums use channel 10.
- The visible overlay canvas uses device-pixel coordinates. Hand points and mouse points are converted to that same coordinate space before wheel hit-testing.
- Preset snapshots are version 6 and must remain plain JSON. Convert harmonic arrays back to `Float32Array` when applying a snapshot. Runtime-only loop record/play positions are never restored.
- MIDI device, channel, transpose, controller, and monitor preferences are device-local rather than patch state. MIDI access always starts disconnected because permission must come from a user gesture.
- `FACTORY_PRESETS` entries are `{patch, meta}` objects. `meta` supplies category, description, and tags; always pass `.patch` into `applySnapshot()`.
- `scopedPreset()` starts from the current version-6 snapshot and selectively replaces fields. Sound scope preserves harmony/rhythm; Groove scope replaces key, wheel, loop, arp, drums, and feel; Visual scope replaces only theme/scene/opacity.
- Keep all optional-control wiring null-safe. A removed panel control previously stopped the entire ES module before boot, which made the wheel appear to be gone.

## Verification checklist

1. Start with **no camera** and confirm the seven-slice wheel appears.
2. Click and hold a slice; confirm chord and volume readouts change and audio sounds.
3. Open Settings; verify oscillator, ADSR, LFO, mixer, arp, drum, visual, chord, and preset controls.
4. Pop out several settings modules; drag, resize, minimize, maximize, overlap/focus, and dock them. Confirm every control stays live and oscillator/envelope canvases redraw at their new width.
5. In Performance, tap a tempo, enable a one-bar count-in and metronome, start with Space, and confirm four count clicks precede the shared downbeat. Test Hold last chord, all bass modes, register/spread revoicing, humanize, and Escape panic/transport stop.
6. Seed each chord-loop progression, edit and audition cells, change 4/8/16-beat lengths, transfer in both directions with the wheel, and confirm loop rests release notes. Arm record from stopped transport and capture mouse/keyboard/hand chords one beat at a time.
7. Run the chord loop with drums and a synced arp. Confirm the top loop chip, selected wheel slice, chord readout, chart highlights, MIDI notes, visuals, and arp harmony change together on beat boundaries.
8. Enable an IAC bus in Audio MIDI Setup, connect it in the MIDI · Logic module, and confirm Test middle C reaches a selected Logic software-instrument track.
9. Hold and change wheel/loop chords; confirm Logic receives voiced note on/off plus CC11 and CC1, channel/octave changes do not leave hanging notes, Panic clears all notes, and Browser audio prevents double monitoring when disabled.
10. Export `.mid` with different loop/drum lengths, swing, humanize, mute/solo, accents, register/spread, and bass modes. Import into Logic and confirm tempo, voiced chord, and GM drum tracks align for the full arrangement.
11. Double-click a `⌁` mixer or envelope label; map it to an LFO and confirm the value badge animates.
12. Map filter cutoff plus Performance register/spread/humanize to hand sources. Confirm their top-left chips appear, show formatted values, dim with no hand, disappear when unmapped/remapped to an LFO, and restore with presets.
13. With a hand-mapped control active, try each visualizer. Confirm the mapped value steers color/intensity and hand motion produces blooms, particle agitation, ripples, mandala movement, or flow bursts as appropriate.
14. Browse all fifteen factory worlds by category, description, Previous/Next, and Surprise me. Confirm every full recall changes key, eight wheel chords, sixteen-beat loop writing, sound, feel, rhythm, mappings, mix, and visual without invalid chord markers or console errors.
15. Test every recall scope. Sound & mix must leave the current wheel/loop/drums intact; Groove & chords must preserve the current timbre/look; Visual must change only theme/scene/opacity. Sound/Visual recall during loop recording must preserve the live record position.
16. Run the guided tour through all eleven interactive tasks; confirm the camera starts only at the hand-routing step and slider arrow keys do not navigate the tour.
17. Exercise 8/16/32-step drum lengths, both 32-step pages, accents, probability, mute/solo, page copy-paste, and pattern tools. Start the drums with a synced arp and chord loop; verify page follow and swing without tempo drift.
18. If camera testing is available, start with camera permission and verify one- and two-hand behavior.

## Known limitations / sensible next work

These are enhancements, not missing parts of the current request:

- The MusicXML reader uses the first part and harmony tags; it is not a full notation renderer.
- Keyboard shortcuts directly address the first ten wheel slices. Slices 11–16 remain available by hand/mouse.
- Presets are stored in browser `localStorage`; there is no cloud sync.
- Chord-loop cells are quarter-note events in 4/4. MIDI export does not include live CC automation, probability outcomes, audio, or oscillator timbre.
- Mobile layout works best in landscape, but the dense settings panel could use a dedicated narrow-screen pass.
- Automated audio assertions are limited by browser autoplay and camera permission policies, so final sound/camera QA still benefits from a human pass.
