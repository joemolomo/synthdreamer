# synthdreamer

A synthwave live-coding library for [Strudel](https://strudel.cc), extending
[Switch Angel's strudel-scripts](https://github.com/switchangel/strudel-scripts)
with synthwave-specific helpers, arpeggiation, gated pads, and preset patches.

Switch Angel's original functions (`blockArrange`, `fill`, and much more) are
included verbatim as the foundation. All new additions appear below them in
`prebake.strudel`.

---

## Loading the prebake file

1. Open [strudel.cc](https://strudel.cc) and go to **Settings → Prebake**.
2. Paste the raw URL of `prebake.strudel` from this repo, or copy-paste the
   file contents directly into the prebake editor.
3. Reload the REPL — all functions and patches are now available globally.

---

## Switch Angel's foundation

The following come from Switch Angel's scripts unchanged:

| Function | Description |
|---|---|
| `blockArrange(patArr, modifiers)` | Arrange patterns by section codes (F/B/R/0) |
| `fill` | Fill gaps between events |
| `ar(...)` | Quick arrange over multiple cycles |
| `track(...)` | Tracker-style arrangement |
| `trancegate` / `tgate` | Rhythmic gating effects |
| `acid` | TB-303-style filter envelope |
| `trancearp` | Arpeggiator with preset patterns |
| `glide` | Polyphonic non-legato glide |
| `humanize` | Timing and velocity randomisation |
| `strum` | Chord strum spread |
| `grab` | Quantise notes to a scale |
| `glitch` | Destructive random parameter mutation |
| and more… | See `prebake.strudel` for the full list |

---

## New functions

### `linnDrum(pat)`

Step-sequencer shorthand using single characters. Each character maps to a
LinnDrum-style sample name and represents one equal-length step across the cycle.

| Char | Sample |
|---|---|
| `k` | kick (`bd`) |
| `s` | snare (`sd`) |
| `r` | rimshot (`rim`) |
| `c` | cowbell (`cb`) |
| `h` | closed hihat (`hh`) |
| `o` | open hihat (`oh`) |
| `.` | rest |

```javascript
// 16-step pattern — kick on 1 & 3, snare on 2 & 4, hats in between
$: linnDrum("k.hhs.hhk.hhs.hh")

// Chain .bank() to swap the sample set
$: linnDrum("k...s...k.r.s..h").bank("RolandTR909")
```

---

### `swArp(notes, style, rate)`

Arpeggiator for synthwave melodic patterns.

| Param | Type | Default | Description |
|---|---|---|---|
| `notes` | `string[]` | — | Note names, e.g. `['c4','eb4','g4','bb4']` |
| `style` | `string` | `'up'` | `'up'` \| `'down'` \| `'updown'` \| `'random'` |
| `rate` | `string` | `'1/16'` | Subdivision, e.g. `'1/16'`, `'1/8'` |

`updown` ascends then descends without repeating the top and bottom notes.
`random` uses Strudel's seeded randomness so the sequence is reproducible.

```javascript
// Rising 16th-note arp on a minor seventh chord
$: swArp(['c4','eb4','g4','bb4'], 'up', '1/16').swLead()

// Up-down arp with external swing
$: swArp(['c4','eb4','g4','bb4'], 'updown', '1/16')
     .swLead()
     .swing(0.55)

// Slower random 8th-note arp through a bass octave
$: swArp(['c3','eb3','g3'], 'random', '1/8').swBass()
```

---

### `gatePad(chordPat, rate, atk, rel)`

Gated pad for the classic synthwave pulsing chord sound. Chops a chord pattern
rhythmically and shapes each pulse with ADSR parameters.

| Param | Type | Default | Description |
|---|---|---|---|
| `chordPat` | `string` | — | Comma-separated notes, e.g. `'c3,eb3,g3,bb3'` or alternating `'<c3,eb3,g3 f3,ab3,c4>'` |
| `rate` | `string` | `'1/8'` | Gate rate, e.g. `'1/8'`, `'1/4'` |
| `atk` | `number` | `0.01` | Attack time in seconds |
| `rel` | `number` | `0.1` | Release/decay time in seconds |

```javascript
// Standard 8th-note gated pad through a four-chord synthwave progression
$: gatePad('<c3,eb3,g3,bb3 f3,ab3,c4,eb4 ab2,c3,eb3,g3 g2,bb2,d3,f3>', '1/8').swSaw()

// Tighter 16th-note gate with very short release
$: gatePad('c3,eb3,g3,bb3', '1/16', 0.005, 0.06).swSaw().room(0.9)

// Slow quarter-note pulse — more atmospheric
$: gatePad('<c3,eb3,g3 ab2,c3,eb3>', '1/4', 0.02, 0.3).swSaw()
```

---

### `swArrange(patArr)`

Arrangement helper with synthwave-specific section codes. A self-contained
alternative to `blockArrange` — use it when you want the section codes below
rather than `blockArrange`'s F/B/R/0 model.

**Section codes:**

| Code | Meaning | What it does |
|---|---|---|
| `F` | Full play | No modification (default "on") |
| `0` | Silence | Pattern muted |
| `I` | Intro | Gain 0.55, LPF 700 Hz, room 0.5 |
| `D` | Drop | Gain 1.05 — full energy |
| `B` | Breakdown | `slow(2)`, gain 0.75 |
| `O` | Outro | Sine fade-out over 8 cycles |

> **Note:** `swArrange` defines its own `B` = breakdown/half-tempo. This is
> intentionally different from `blockArrange`'s `B` = backwards. Use
> `blockArrange` if you need reverse playback.

```javascript
const kick  = s("bd").bank("RolandTR909")
const pad   = gatePad('<Cm7 Fm7 Ab7 Gm7>', '1/8').swSaw()
const lead  = swArp(['c4','eb4','g4','bb4'], 'up', '1/16').swLead()
const bass  = note("<c2 eb2 ab2 g2>").swBass()

$: swArrange([
     [kick,  "<0 I I D D D D B B O>"],
     [pad,   "<I I I D D D D B B O>"],
     [lead,  "<0 0 I D D D D 0 0 O>"],
     [bass,  "<0 0 I D D D D B B O>"],
   ])
```

---

## Preset patches

Reusable pattern fragments that define synthwave synth sounds. Assign notes
or chords on top using `.set.out()`.

### `swSaw`

Detuned supersaw pad — warm, wide stereo image, heavy reverb.

```javascript
// Slow chord pad
$: note("<Cm7 Fm7 Ab7 Gm7>").swSaw().slow(4)

// Gated version
$: gatePad('<Cm7 Fm7 Ab7 Gm7>', '1/8').swSaw()
```

### `swLead`

Clean triangle lead — bright with subtle chorus via delay and light vibrato.
Best for arps and melodic lines.

```javascript
$: swArp(['c5','eb5','g5','bb5'], 'up', '1/16').swLead()
```

### `swBass`

Gritty sub bass — tight attack, slight harmonic distortion via waveshaping.

```javascript
$: note("c2 ~ c2 ~ eb2 ~ f2 ~").swBass()

// Pair with kick for punch
$: stack(
     linnDrum("k...s...k...s..."),
     note("c2 ~ ~ ~ ~ ~ eb2 ~").swBass()
   )
```

---

## Combining everything

```javascript
const kick   = linnDrum("k...s...k...s...")
const hats   = linnDrum("..hh..hh..hh..hh")
const pad    = gatePad('<Cm7 Fm7 Ab7 Gm7>', '1/8').swSaw()
const melody = swArp(['c5','eb5','g5','bb5'], 'updown', '1/16').swLead()
const bass   = note("<c2 eb2 ab2 g2>").swBass()

$: swArrange([
     [kick,   "<0 I D D D D D B B O>"],
     [hats,   "<0 0 D D D D D 0 0 O>"],
     [pad,    "<I I D D D D D B B O>"],
     [melody, "<0 0 D D D D D 0 0 O>"],
     [bass,   "<0 I D D D D D B B O>"],
   ])
```

---

## Credits

Switch Angel's [strudel-scripts](https://github.com/switchangel/strudel-scripts)
forms the entire foundation of this library. The core utilities (`blockArrange`,
`fill`, `ar`, `track`, `trancearp`, `tgate`, `acid`, `grab`, `glide`, `strum`,
`humanize`, and more) are her work, reproduced here with gratitude.

Switch Angel — [github.com/switchangel](https://github.com/switchangel)

Glossing — several functions (noted in source) are from
[Glossing's Strudel Scripts](https://codeberg.org/glossing/Strudel_Scripts).
