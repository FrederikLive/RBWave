Below is the implementation plan I would hand directly to the coding agent. I’ve made the first phase deliberately validation-heavy because the biggest technical risk is not drawing the waveform—it is getting reliable, fast PCM analysis of every audio format Live accepts without blocking Live.

# SpectraWave

## Rekordbox-style Multicolor Waveform for Ableton Live

**Document:** Implementation Plan
**Working product name:** SpectraWave
**Initial platform:** Ableton Live + Max for Live
**Primary language:** TypeScript / JavaScript
**UI:** Max `v8ui` + MGraphics
**Background analysis:** Node for Max
**Target:** Ableton Live 12.x, with architecture compatible with Live 11+ where practical
**Document status:** Build specification

---

# 1. Project objective

Build a Max for Live Audio Effect that displays an analyzed, multicolor waveform for Ableton audio clips.

The initial visual model should resemble the functional concept of Rekordbox's 3-band waveform:

* low-frequency energy displayed in blue
* mid-frequency energy displayed in amber/orange
* high-frequency energy displayed in white
* symmetrical waveform around a horizontal center line
* scrolling detailed waveform
* stationary or centered playhead
* complete-track overview
* beat/bar grid
* clip start/end/loop indication
* accurate synchronization to Ableton's playback
* correct visualization of warped clips

This project must **not attempt to alter Ableton's built-in waveform renderer**.

It must instead provide its own waveform interface inside a Max for Live device and, later, optionally a larger detachable display.

The first release should behave as a visualization device and must not intentionally modify the audio passing through it.

---

# 2. Core product requirements

The device must eventually support four principal operating modes:

### Selected Clip

Follow the audio clip currently displayed in Ableton's Clip Detail View.

### Playing Clip

Follow the clip currently playing on the relevant track.

### Device Track

Follow the clip associated with the track containing the SpectraWave device.

### Manual

Allow an audio file to be loaded manually when Live cannot provide a usable clip reference.

The MVP should implement **Selected Clip first**.

Ableton exposes the currently displayed clip as:

`live_set view detail_clip`

through the Live Object Model. The API also exposes the clip's source-file path, sample rate, sample length, playback position and warp markers. ([Cycling '74 Documentation][1])

---

# 3. Non-goals for version 1

Do not attempt these before the basic waveform is stable:

* editing Ableton warp markers
* replacing Live's native waveform
* stem separation
* vocal detection
* key detection
* BPM detection
* beat detection independent of Ableton
* MIDI waveform support
* cloud processing
* telemetry
* online services
* Rekordbox file/database parsing
* direct Rekordbox integration
* GPU shader implementation
* VST3 version
* AU version
* machine-learning features

These may become later phases.

The MVP succeeds when SpectraWave accurately displays Ableton audio clips as a multicolor waveform and follows playback.

---

# 4. Fundamental architecture

Use the following separation of responsibilities:

```text
                   ABLETON LIVE
                       │
                       │ Live Object Model
                       ▼
               ┌─────────────────┐
               │   LOM ADAPTER   │
               │                 │
               │ selected clip   │
               │ file path       │
               │ warp markers    │
               │ loop markers    │
               │ play position   │
               └────────┬────────┘
                        │
                        ▼
               ┌─────────────────┐
               │ CLIP CONTROLLER │
               │                 │
               │ validates clip  │
               │ state machine   │
               │ change tracking │
               └────────┬────────┘
                        │
            ┌───────────┴────────────┐
            │                        │
            ▼                        ▼
 ┌───────────────────┐      ┌──────────────────┐
 │ ANALYSIS SERVICE  │      │   WARP MAPPER    │
 │ Node for Max      │      │                  │
 │                   │      │ beat → source    │
 │ decode audio      │      │ source → beat    │
 │ band analysis     │      │ interpolation    │
 │ normalization     │      └────────┬─────────┘
 │ cache creation    │               │
 └─────────┬─────────┘               │
           │                         │
           └────────────┬────────────┘
                        ▼
                ┌───────────────┐
                │ WAVEFORM DATA │
                │               │
                │  LOW          │
                │  MID          │
                │  HIGH         │
                │  PEAK         │
                │  mip levels   │
                └───────┬───────┘
                        │
                        ▼
              ┌────────────────────┐
              │     RENDERER       │
              │                    │
              │ v8ui + MGraphics   │
              │ overview           │
              │ scrolling detail   │
              │ playhead           │
              │ beat grid          │
              └────────────────────┘
```

Each box should have a clearly defined interface.

Do not create one giant JavaScript file.

---

# 5. Why Node for Max should handle analysis

The actual frequency analysis should not execute synchronously on the main Max UI scheduler.

Use `node.script` for long-running analysis tasks.

Cycling '74 documents that `node.script` launches a separate Node process and can therefore run asynchronously and on a separate CPU core from Max. It also provides ordinary file I/O and npm package access. ([Cycling '74 Documentation][2])

That gives the architecture:

```text
MAX / LIVE THREAD
     │
     │ analyze(path)
     ▼
NODE PROCESS
     │
     ├── decode
     ├── FFT/filter analysis
     ├── normalization
     ├── mipmaps
     └── cache
     │
     ▼
analysis_ready(cache-id)
```

The UI must remain responsive during analysis.

---

# 6. Critical technical risk: audio decoding

This must be investigated before the full analyzer is written.

Current Max documentation explicitly documents `sfplay~` support for formats including:

* AIFF
* WAV
* MP3
* OGG
* FLAC
* M4A
* CAF
* WF64
* WAVE64

([Cycling '74 Documentation][3])

However, the documented `buffer~` reading interface is more restrictive and explicitly describes AIFF/WAVE/AU-style loading rather than guaranteeing every compressed format. ([Cycling '74 Documentation][4])

Therefore:

**Do not design the final analyzer around `buffer~` until compatibility is experimentally verified.**

The agent must implement a decoder abstraction.

```typescript
interface AudioDecoder {
    supports(path: string): Promise<boolean>;
    probe(path: string): Promise<AudioFileInfo>;
    decode(path: string): AsyncIterable<PcmChunk>;
}
```

Where:

```typescript
interface PcmChunk {
    sampleRate: number;
    channels: number;
    frames: number;
    data: Float32Array[];
}
```

Potential implementations should be evaluated in Phase 0:

```text
Decoder A
Pure Node / JS/WASM decoder

Decoder B
Max-compatible mechanism

Decoder C
FFmpeg-backed decoder

Decoder D
OS/native decoding if practical
```

Do not commit to FFmpeg or any third-party decoding library without checking:

* Windows support
* macOS support
* Max for Live packaging
* redistribution licensing
* device freeze/export behavior
* binary size
* Intel/ARM Mac compatibility
* Node version compatibility

For early development, WAV should be sufficient.

Compressed-format support becomes mandatory before release.

---

# 7. Source repository structure

Create this structure before implementation:

```text
SpectraWave/
│
├── README.md
├── IMPLEMENTATION_PLAN.md
├── CHANGELOG.md
├── LICENSE
├── package.json
├── tsconfig.json
├── .gitignore
│
├── docs/
│   ├── ARCHITECTURE.md
│   ├── ANALYSIS.md
│   ├── WARP_MAPPING.md
│   ├── UI.md
│   ├── TESTING.md
│   ├── DECISIONS.md
│   ├── STATUS.md
│   └── HANDOFF.md
│
├── max/
│   ├── SpectraWave.amxd
│   ├── patchers/
│   │   ├── spectrawave-main.maxpat
│   │   ├── lom-controller.maxpat
│   │   └── audio-passthrough.maxpat
│   │
│   └── code/
│       ├── lom-adapter.js
│       ├── transport-controller.js
│       ├── renderer.js
│       └── ui-controller.js
│
├── src/
│   ├── analysis/
│   │   ├── analyzer.ts
│   │   ├── fft.ts
│   │   ├── bands.ts
│   │   ├── normalize.ts
│   │   ├── pyramid.ts
│   │   └── types.ts
│   │
│   ├── decoder/
│   │   ├── decoder.ts
│   │   ├── wav-decoder.ts
│   │   └── decoder-registry.ts
│   │
│   ├── cache/
│   │   ├── cache-manager.ts
│   │   ├── cache-key.ts
│   │   └── cache-format.ts
│   │
│   ├── warp/
│   │   ├── warp-map.ts
│   │   └── interpolation.ts
│   │
│   ├── renderer/
│   │   ├── viewport.ts
│   │   ├── downsample.ts
│   │   └── colors.ts
│   │
│   └── node/
│       └── service.ts
│
├── tests/
│   ├── analysis/
│   ├── warp/
│   ├── cache/
│   ├── fixtures/
│   └── integration/
│
└── tools/
    ├── generate-test-audio.ts
    └── inspect-cache.ts
```

Adjust the precise Max patch organization if required by Max, but preserve separation of concerns.

---

# 8. Data contracts

The project must define stable internal contracts before UI development.

## ClipDescriptor

```typescript
interface ClipDescriptor {
    liveId: number;

    name: string;

    filePath: string;

    isAudioClip: boolean;

    sampleRate: number;

    sampleLength: number;

    warping: boolean;

    warpMarkers: WarpMarker[];

    startMarker: number;
    endMarker: number;

    loopStart: number;
    loopEnd: number;
    looping: boolean;

    isArrangementClip: boolean;
    isSessionClip: boolean;
}
```

## WarpMarker

```typescript
interface WarpMarker {
    sampleTime: number;
    beatTime: number;
}
```

Ableton defines warp markers using source `sample_time` in seconds paired with `beat_time`. Ableton's documented conversion itself uses interpolation between neighboring warp markers. ([Cycling '74 Documentation][5])

## AnalysisDescriptor

```typescript
interface AnalysisDescriptor {
    schemaVersion: number;

    source: {
        path: string;
        fileSize: number;
        modifiedTime: number;
        sampleRate: number;
        channels: number;
        durationSeconds: number;
    };

    algorithm: {
        version: string;
        analysisRate: number;
        fftSize: number;
        hopSize: number;
        window: string;

        bands: {
            low: [number, number];
            mid: [number, number];
            high: [number, number];
        };
    };

    frameCount: number;

    low: Uint8Array;
    mid: Uint8Array;
    high: Uint8Array;

    peak?: Uint8Array;

    pyramid: WaveformLevel[];
}
```

Recommended initial frequency ranges:

```text
LOW
20 Hz – 250 Hz

MID
250 Hz – 4 kHz

HIGH
4 kHz – Nyquist
```

These are SpectraWave design parameters, not claimed Rekordbox specifications.

Put them in configuration rather than hardcoding them throughout the project.

---

# 9. Analysis resolution

Initial target:

```text
150 analysis frames / second
```

At five minutes:

```text
5 × 60 × 150
=
45,000 waveform frames
```

Three 8-bit bands require approximately:

```text
45,000 × 3
=
135,000 bytes
```

before metadata and mip levels.

This is tiny enough that complete analyzed waveform data can remain in memory.

Do not keep decoded raw PCM in memory once analysis is complete.

---

# 10. Analysis algorithm V1

Start with STFT analysis because it gives us future flexibility.

For each window:

```text
PCM
 ↓
stereo → mono energy combination
 ↓
Hann window
 ↓
FFT
 ↓
power spectrum
 ↓
sum bins into frequency bands
 ↓
LOW / MID / HIGH
 ↓
compression
 ↓
normalization
 ↓
8-bit waveform value
```

Recommended starting parameters:

```text
FFT size        2048
Window          Hann
Analysis rate   ~150 Hz
Band metric     RMS/power
```

The exact hop size should be derived from sample rate:

```text
hop ≈ sampleRate / 150
```

Do not assume 44.1 kHz.

---

# 11. Stereo handling

Do not simply analyze the left channel.

For each corresponding L/R sample, begin with:

```text
mono = sqrt((L² + R²) / 2)
```

or use equivalent energy combination during spectral analysis.

The purpose is waveform visualization, not reconstructing the audio.

Later versions may offer:

```text
Combined
Split L/R
Mid/Side
```

V1 only requires combined.

---

# 12. Amplitude compression

Raw spectral power will make typical mastered music appear nearly solid.

Use perceptual compression.

Candidate transform:

```text
compressed = log1p(k * energy)
```

Then calculate normalization across the entire song.

Prefer robust percentile normalization:

```text
reference = percentile(values, 99.5)
normalized = clamp(value / reference, 0, 1)
```

Then optionally:

```text
visual = normalized ^ gamma
```

Initial:

```text
gamma = 0.65
```

All constants must be centralized in:

```text
analysis-config.ts
```

The renderer must not contain analysis magic numbers.

---

# 13. Multi-resolution waveform pyramid

This is important.

Do not make the renderer aggregate tens of thousands of analysis samples every frame.

Generate waveform mip levels.

Example:

```text
LEVEL 0
150 columns/sec

LEVEL 1
75 columns/sec

LEVEL 2
37.5 columns/sec

LEVEL 3
18.75 columns/sec

LEVEL 4
9.375 columns/sec

...
```

Each higher level combines two neighboring frames.

For each band retain:

```text
maximum energy
```

or a weighted maximum/RMS combination.

For waveform display, peak-preserving aggregation is preferable because transients should remain visible at low zoom.

Conceptually:

```text
L0: ▂ ▆ ▃ █ ▅ ▂ █ ▃

L1:   ▆   █   ▅   █

L2:      █       █

L3:          █
```

The renderer selects the level closest to one source waveform column per display pixel.

---

# 14. Cache format

Do not re-analyze songs on every selection.

Use persistent analysis caching.

Suggested cache identity:

```text
absolute path
+
file size
+
modified timestamp
+
analysis algorithm version
```

Hash this into:

```text
SHA-256
```

Cache directory:

### Windows

```text
%LOCALAPPDATA%/SpectraWave/Cache/
```

### macOS

```text
~/Library/Caches/SpectraWave/
```

Each cache item:

```text
<hash>.json
<hash>.bin
```

JSON:

```json
{
  "schemaVersion": 1,
  "source": {},
  "algorithm": {},
  "frameCount": 45000,
  "levels": []
}
```

BIN:

```text
LOW MID HIGH LOW MID HIGH ...
```

Do not store massive integer arrays in JSON.

---

# 15. Cache invalidation

A cache entry becomes invalid if:

```text
file size changed

OR

modification timestamp changed

OR

analysis algorithm version changed

OR

cache schema version changed
```

Later add optional partial-content hashes for moved files.

V1 does not need content-addressable file relocation.

---

# 16. Live Object Model integration

The primary path is:

```text
live_set view detail_clip
```

Observe this object.

When the object ID changes:

```text
detail_clip_changed
        │
        ▼
is valid ID?
   │        │
  no       yes
   │        │
 empty      ▼
 state   is_audio_clip?
             │
        ┌────┴────┐
        no       yes
        │         │
       empty      ▼
             fetch metadata
```

Fetch at minimum:

```text
name
file_path
sample_rate
sample_length
is_audio_clip
is_arrangement_clip
is_session_clip
warping
warp_markers
playing_position
start_marker
end_marker
loop_start
loop_end
looping
```

These properties are available through Ableton's Clip Live Object Model. ([Cycling '74 Documentation][6])

Do not re-query static clip metadata every render frame.

---

# 17. Observers

Create observers for:

```text
detail_clip

warp_markers

warping

start_marker

end_marker

loop_start

loop_end

looping

playing_status
```

When warp markers change, Ableton's observer emits a bang rather than the entire marker collection, so fetch the full warp-marker dictionary again. ([Cycling '74 Documentation][5])

The event sequence should be:

```text
warp_markers changed
       ↓
invalidate warp map
       ↓
read complete markers
       ↓
build sorted mapping arrays
       ↓
refresh display
```

Waveform spectral analysis does **not** need to run again when warp markers change.

Only geometry changes.

---

# 18. Warp mapping

This is one of the most important pure functions in the application.

Implement:

```typescript
beatToSourceSeconds(beat: number): number
```

and:

```typescript
sourceSecondsToBeat(seconds: number): number
```

For two markers:

```text
M0:
beat = B0
sample = S0

M1:
beat = B1
sample = S1
```

For beat `B`:

```text
r = (B - B0) / (B1 - B0)

S = S0 + r × (S1 - S0)
```

This matches the interpolation approach documented by Ableton. ([Cycling '74 Documentation][5])

Use binary search to find neighboring markers.

Complexity:

```text
O(log n)
```

Never linearly scan every warp marker during every pixel calculation.

---

# 19. Warp-map precomputation

Store:

```typescript
class WarpMap {
    beatTimes: Float64Array;
    sampleTimes: Float64Array;

    beatToSeconds(beat: number): number;
    secondsToBeat(seconds: number): number;
}
```

Validate markers on creation:

```text
beatTimes monotonic

sampleTimes monotonic

minimum 2 usable markers where required

finite numeric values only
```

If malformed:

```text
fall back safely

show status warning

do not crash renderer
```

---

# 20. Unwarped audio

Ableton reports `playing_position` differently for warped and unwarped audio:

* warped audio → beats
* unwarped audio → seconds

([Cycling '74 Documentation][5])

Therefore transport logic must explicitly branch:

```typescript
if (clip.warping) {
    sourceSeconds =
        warpMap.beatToSeconds(playingPosition);
} else {
    sourceSeconds =
        playingPosition;
}
```

Do not assume all audio clips are warped.

---

# 21. Renderer technology

Use:

```text
v8ui
+
MGraphics
```

rather than the legacy JS engine for new UI work.

`v8ui` provides modern JavaScript with custom visual output through MGraphics. ([Cycling '74 Documentation][7])

MGraphics supports custom paths, shapes, surfaces, text and offscreen rendering. ([Cycling '74 Documentation][8])

Create the renderer as a largely pure component receiving:

```typescript
interface RenderState {
    width: number;
    height: number;

    waveform: WaveformData;

    viewport: Viewport;

    sourcePositionSeconds: number;

    clip: ClipDescriptor;

    theme: Theme;
}
```

It should not directly call the Live API.

---

# 22. Waveform drawing order

Draw in this order:

```text
1. background

2. grid

3. LOW waveform

4. MID waveform

5. HIGH waveform

6. clip boundaries

7. loop region

8. cue/warp indicators if enabled

9. playhead

10. text/status
```

Recommended initial palette:

```text
Background
#111318

Low
electric/deep blue

Mid
amber/orange

High
near-white
```

Do not copy Rekordbox graphic assets, branding, typography or proprietary UI components.

This is a functional 3-band visualization concept.

---

# 23. Layer model

For each x-coordinate calculate:

```text
lowHeight
midHeight
highHeight
```

Draw symmetric around center:

```text
       high
        █
      █ █ █
    █ █ █ █ █
────────────────── center
    █ █ █ █ █
      █ █ █
        █
```

Prefer filled vertical bars or continuous polygons depending on performance.

Start with vertical columns because they are simpler and visually appropriate.

Once stable, compare:

```text
vertical line renderer

filled polygon renderer
```

Measure performance before choosing final implementation.

---

# 24. Basic UI layout

Initial compact device:

```text
┌───────────────────────────────────────────────────────┐
│ SPECTRAWAVE   [Selected ▼] [3 BAND ▼] [Follow ●]    │
├───────────────────────────────────────────────────────┤
│                                                       │
│     ▂▄▇██▅▃▆████▃▂▅██▇▅▄▆██████▅▂                   │
│                       │                               │
│                       │                               │
│                     PLAY                              │
│                                                       │
├───────────────────────────────────────────────────────┤
│ 1.1      5.1       9.1       13.1      17.1          │
└───────────────────────────────────────────────────────┘
```

Do not overload MVP UI.

Controls:

```text
Source
Selected / Playing / Track / Manual

Display
3 Band / Classic

Follow
On / Off

Zoom
1 / 2 / 4 / 8 / 16 / 32 bars

Overview
toggle
```

---

# 25. Detailed waveform viewport

Define viewport in beats when warped:

```typescript
interface BeatViewport {
    startBeat: number;
    endBeat: number;
}
```

For unwarped clips:

```typescript
interface TimeViewport {
    startSeconds: number;
    endSeconds: number;
}
```

Do not mix these implicitly.

Introduce a common abstraction:

```typescript
type Viewport =
    | BeatViewport
    | TimeViewport;
```

---

# 26. Follow mode

When Follow is enabled:

```text
current playback position
           │
           ▼
place at 50% horizontal position
```

Thus:

```text
────────────────────────────────────
              │
              │ PLAYHEAD
              │
────────────────────────────────────
             50%
```

Allow later options:

```text
25%
50%
75%
```

MVP uses 50%.

---

# 27. Zoom

For warped material, zoom should be musical.

Recommended:

```text
1 bar
2 bars
4 bars
8 bars
16 bars
32 bars
64 bars
Full Clip
```

Respect clip time signature if practical.

MVP can assume four beats/bar for the initial visual grid if necessary, but the final release should respect clip/song signatures.

Document any interim assumption.

---

# 28. Grid drawing

At broad zoom:

```text
BAR
│
│   beat
│   │
1 . 2 . 3 . 4
```

Grid hierarchy:

```text
bar line       strongest
beat line      medium
subdivision    faint
```

Grid density must reduce automatically at large zoom levels.

Never attempt to draw thousands of grid lines.

---

# 29. Complete-track overview

Create a separate overview strip:

```text
FULL SONG

▂▄████▆▃▄▅███▆▄▂▇████▅▄███▆▄▃█████
              ▲
              playback
```

Overlay the detailed viewport:

```text
▂▄████▆▃▄▅███▆▄▂▇████▅▄███▆▄▃█████
        └──────────────┘
          visible area
```

Later allow clicking overview to seek only if this can be implemented safely without creating unexpected transport behavior.

Do not implement seeking in MVP.

---

# 30. Rendering optimization

The expensive part is the static waveform.

Separate:

```text
STATIC LAYER

waveform
grid
loops
background

DYNAMIC LAYER

playhead
hover
status
```

Where practical, cache static graphics using an offscreen MGraphics surface.

MGraphics supports offscreen contexts and images that can be reused rather than reconstructed continuously. ([Cycling '74 Documentation][8])

For static/non-follow view:

```text
render waveform once

then only repaint playhead
```

For scrolling follow mode the waveform must move, so use the appropriate pyramid level and only calculate visible columns.

---

# 31. Transport refresh

Target initial visual refresh:

```text
30 FPS
```

Optional:

```text
60 FPS
```

Do not hammer the Live API 60 times per second with all clip properties.

Transport strategy:

```text
Live observer/poll
      ↓
latestPlaybackPosition
      ↓
UI timer
      ↓
render frame
```

Metadata queries should occur only when state changes.

---

# 32. Playing Clip mode

Implement only after Selected Clip mode works.

For Session View, Live exposes:

```text
playing_slot_index
```

on tracks.

For Arrangement View, tracks expose:

```text
arrangement_clips
```

since Live 11. ([Cycling '74 Documentation][9])

The agent should implement a TrackClipResolver abstraction rather than putting Session and Arrangement logic directly into the UI.

```typescript
interface TrackClipResolver {
    getCurrentClip(): Promise<ClipReference | null>;
}
```

Strategies:

```text
SelectedClipResolver

SessionPlayingClipResolver

ArrangementPlayingClipResolver
```

---

# 33. Arrangement playing clip resolution

For the Device Track:

```text
this_device
   ↓
canonical parent
   ↓
Track
   ↓
arrangement_clips
```

Find clips whose arrangement interval contains the song's playback beat.

Do not scan the list every render frame.

Cache:

```text
sorted arrangement intervals
```

Then binary search by song time.

Refresh the list only when Ableton reports arrangement clip changes.

---

# 34. Device audio behavior

SpectraWave should be an Audio Effect.

Audio path:

```text
plugin~ L/R
   │
   └─────────────► plugout~ L/R
```

No DSP modification.

If Max requires internal objects for proper device operation, preserve exact audio.

Test using:

```text
original render
vs
SpectraWave render
```

Expected difference:

```text
digital null
```

or zero within the platform's unavoidable numerical behavior.

SpectraWave must introduce no intentional gain, filtering or latency.

---

# 35. UI states

The UI needs explicit states.

```typescript
enum DeviceState {
    NoClip,
    MidiClip,
    MissingFile,
    NeedsAnalysis,
    Analyzing,
    Ready,
    Error
}
```

Display appropriate messages.

### No clip

```text
Select an audio clip
```

### MIDI clip

```text
SpectraWave displays audio clips only
```

### Missing file

```text
Source audio file is unavailable
```

### Analyzing

```text
Analyzing waveform… 42%
```

### Error

```text
Waveform analysis failed
[Retry]
```

Do not show raw exceptions to ordinary users.

Write detailed errors to the Max console/log.

---

# 36. Analysis progress

Node analysis should report:

```text
0–100%
```

Messages:

```text
analysis:start

analysis:progress

analysis:complete

analysis:error

analysis:cancelled
```

Example:

```json
{
  "type": "analysis:progress",
  "jobId": "abc123",
  "progress": 0.64
}
```

---

# 37. Job cancellation

Clip selection can change while analysis is running.

Therefore:

```text
Clip A selected
     ↓
Analyze A
     ↓
user selects B
     ↓
CANCEL A
     ↓
Analyze B
```

Every analysis operation receives:

```text
jobId
```

Results from obsolete job IDs must be discarded.

Never allow:

```text
A analysis finishes
↓
UI currently showing B
↓
A waveform accidentally replaces B
```

---

# 38. Analysis queue

Initial maximum concurrent analyses:

```text
1
```

This protects CPU usage.

Later:

```text
1 foreground
1 background
```

may be evaluated.

Do not make analysis concurrency configurable before profiling.

---

# 39. Error handling

No uncaught exception should terminate the Node service.

Handle:

```text
file deleted

permission denied

unsupported codec

corrupt audio

zero-length file

NaN audio samples

unexpected channel count

cache corruption

cache schema mismatch

Live object ID becoming invalid

device deleted during analysis

clip replaced while analyzing
```

Cache corruption should cause:

```text
delete/ignore cache
→
reanalyze
```

not a fatal error.

---

# 40. Test audio generator

Create `tools/generate-test-audio.ts`.

Generate deterministic fixtures:

```text
20 Hz sine

60 Hz sine

120 Hz sine

1 kHz sine

8 kHz sine

white noise

pink noise

impulse train

kick-like signal

sweep 20 Hz → 20 kHz

silence

clipped waveform
```

Use these to verify band classification.

Examples:

```text
60 Hz
LOW should dominate

1 kHz
MID should dominate

8 kHz
HIGH should dominate
```

Turn these into automated tests.

---

# 41. Warp-map tests

Create deterministic marker sets.

Example:

```text
sample 0s    → beat 0
sample 10s   → beat 20
sample 20s   → beat 30
```

Tests:

```text
beat 10
should map to 5 s

beat 25
should map to 15 s
```

Test reverse conversion as well.

Test:

```text
exact marker

between markers

before first marker

after last marker

duplicate marker rejection

invalid ordering
```

This subsystem must have near-100% unit-test coverage because visual synchronization depends on it.

---

# 42. Performance targets

Release targets:

### Cached clip load

```text
< 250 ms
```

for a typical song after cache lookup.

### UI

```text
30 FPS minimum while following playback
```

on a normal modern desktop.

### Idle CPU

Aim for:

```text
< 2%
```

for a loaded but stopped device.

### Playing visualization

Aim for:

```text
< 5%
```

where realistically achievable.

### Resident analyzed waveform memory

Typical song:

```text
< 10 MB
```

### Raw PCM

Release PCM immediately after analysis.

Do not retain decoded source audio.

Analysis performance must be measured before hard requirements are tightened.

---

# 43. Phase 0 — Technical feasibility spikes

**Do not start polished UI before completing this phase.**

## Spike A — Live clip metadata

Create a minimal Max device that prints:

```text
detail clip ID
clip name
file path
sample rate
sample length
warping
warp markers
playing position
```

Pass when:

* selecting another clip updates values
* MIDI clips are detected
* no-clip selection does not error
* warped clip returns markers
* unwarped clip returns correct playback units

## Spike B — transport tracking

Display:

```text
playing_position
```

live at approximately 30 FPS.

Determine whether observer updates are sufficient or a throttled poll is required.

Record result in:

```text
docs/DECISIONS.md
```

## Spike C — v8ui renderer

Render synthetic:

```text
low
mid
high
```

arrays as colored waveform layers.

Target 60 FPS with ~2,000 display columns.

## Spike D — decoding

Test:

```text
WAV
AIFF
MP3
FLAC
M4A
OGG
```

on both Windows and macOS where environments are available.

Determine:

```text
which decoder backend

what dependencies

what packaging requirements
```

## Spike E — Node for Max

Verify:

```text
Max → Node message

Node → Max message

file read

background analysis

job cancellation
```

## Phase 0 gate

Only proceed when:

```text
Live metadata works

Node communication works

waveform renderer works

at least WAV analysis works

decoder strategy is documented
```

---

# 44. Phase 1 — Static waveform MVP

Goal:

Select an audio clip and display its full-track 3-band waveform.

Implement:

```text
detail_clip observer

file path retrieval

WAV decoder

analysis engine

normalization

cache

static v8ui waveform
```

No playback scrolling required yet.

Acceptance criteria:

```text
select WAV clip

analysis begins automatically

progress visible

3-band waveform appears

reselect clip

waveform loads from cache

change to another file

correct waveform appears
```

---

# 45. Phase 2 — Playback synchronization

Implement:

```text
playing_position

playhead rendering

unwarped playback

warped playback

warp map interpolation
```

Acceptance test:

Create a file containing sharp impulses at known times.

Warp several impulses onto obvious bar positions.

During playback the playhead should visually cross each impulse when it is heard.

Tolerance target:

```text
≤ one rendered waveform pixel
```

where achievable.

---

# 46. Phase 3 — Detailed scrolling waveform

Implement:

```text
centered playhead

follow mode

bar zoom levels

dynamic viewport

grid

loop display

start/end display

mip-level selection
```

Acceptance:

```text
playback remains smooth

zoom can change while playing

no UI freeze

waveform remains aligned after zoom

warped areas visibly stretch/compress
```

---

# 47. Phase 4 — Full-track overview

Implement:

```text
overview waveform

current-position indicator

visible-range indicator
```

Do not implement seeking yet.

Acceptance:

Overview remains stable while detailed waveform scrolls.

---

# 48. Phase 5 — Playing Clip / Track mode

Implement:

```text
source mode selector

Selected

Playing

Track
```

Use:

```text
Track.playing_slot_index
```

for Session clips.

Use:

```text
Track.arrangement_clips
```

for Arrangement resolution. Ableton exposes arrangement clip IDs from the Track object. ([Cycling '74 Documentation][9])

Acceptance:

Switch between multiple clips during playback without stale analysis being shown.

---

# 49. Phase 6 — Format compatibility

Complete support for the common Ableton workflows:

```text
WAV

AIFF

MP3

FLAC

M4A

OGG
```

Do not claim support until each format has been tested.

Create:

```text
tests/FORMAT_MATRIX.md
```

Columns:

```text
Format
Windows
macOS Intel
macOS ARM
Decode
Analyze
Cache
Reload
```

---

# 50. Phase 7 — UI polish

Only now add:

```text
theme

tooltips

hover

better typography

status animations

settings menu

overview styling

smooth scrolling

optional 60 FPS
```

Do not let visual polishing destabilize synchronization.

---

# 51. Phase 8 — Expanded visualization modes

After 3-band is stable:

```text
3 BAND

CLASSIC

RGB

SPECTRAL
```

Classic:

single-color amplitude waveform.

RGB:

```text
low  → red
mid  → green
high → blue
```

Spectral:

frequency-dependent continuous coloring.

The analysis architecture should already support these without reworking clip synchronization.

---

# 52. Phase 9 — Optional musical-analysis overlays

Only after the foundational product is finished:

Potential overlays:

```text
kick likelihood

snare likelihood

bass regions

vocal regions

energy curve

phrase boundaries

drops

breakdowns
```

Each must be independently toggleable.

Never mix experimental detection into the core waveform cache schema without versioning it.

---

# 53. Future detached window

Investigate a larger floating interface later.

Potential layout:

```text
┌─────────────────────────────────────────────────────┐
│ SpectraWave                                         │
├─────────────────────────────────────────────────────┤
│ FULL SONG OVERVIEW                                  │
│ ▃▄████▅▂▃██████▅▄▄█████▂▆███████                 │
│                     ▲                               │
├─────────────────────────────────────────────────────┤
│                                                     │
│       LARGE SCROLLING 3-BAND WAVEFORM              │
│                                                     │
│                       │                             │
│                       │ PLAYHEAD                    │
│                                                     │
├─────────────────────────────────────────────────────┤
│ 1.1      5.1      9.1     13.1     17.1            │
└─────────────────────────────────────────────────────┘
```

Do not make this a blocker for initial release.

---

# 54. Development methodology for autonomous agent

The agent should operate continuously through phases without waiting for approval after every successful component.

After finishing a task:

```text
run tests

fix failures

update STATUS.md

update HANDOFF.md

continue to next unblocked task
```

Only stop for human input when:

```text
a required external installation needs user action

Ableton/Max manual testing is absolutely required

a licensing decision changes product distribution

a platform-specific problem cannot be reproduced

requirements conflict irreconcilably
```

Do not stop merely because:

```text
one file was completed

one phase subsection was completed

a minor design decision exists

tests passed
```

Choose a reasonable implementation and document the decision.

---

# 55. STATUS.md

Keep continuously updated:

```markdown
# Status

## Current Phase

Phase 2 — Playback Synchronization

## Completed

- LOM detail_clip observer
- WAV decoding
- FFT analysis
- cache v1
- static renderer

## In Progress

- WarpMap interpolation

## Next

- playhead synchronization
- warped impulse test

## Blockers

None

## Known Issues

- M4A decoder not selected yet
```

---

# 56. HANDOFF.md

This file must allow another coding agent to continue immediately.

Maintain:

```markdown
# Agent Handoff

## Current Goal

Implement warped clip playback synchronization.

## Last Completed Work

...

## Architecture State

...

## Commands

npm install
npm test
npm run build

## Important Files

src/warp/warp-map.ts
max/code/lom-adapter.js

## Tests

142 passing
0 failing

## Current Problems

...

## Next Exact Action

Implement sourceSecondsToBeat() binary search.

## Do Not Change

Cache schema v1 until Phase 6.
```

Update this before context/token exhaustion and at the end of every major phase.

---

# 57. DECISIONS.md

Every architectural decision that would otherwise be forgotten goes here.

Format:

```markdown
## ADR-004 — Use Node for analysis

Status: Accepted

Reason:
Avoid blocking Max UI and allow filesystem/npm access.

Alternatives:
Main-thread JS
MSP-only analysis

Consequences:
Node process must be packaged with device.
```

Use ADR-style sequential IDs.

---

# 58. Automated testing requirement

No pure-data subsystem should depend on Ableton for testing.

The following must run from ordinary command line:

```text
analysis

normalization

waveform pyramid

warp interpolation

cache keys

cache serialization

viewport math
```

Use built-in Node testing or another lightweight framework.

Avoid an unnecessarily complex test stack.

---

# 59. Live integration test harness

Create a mock LOM layer.

Example:

```typescript
const mockClip = {
    filePath: "/fixtures/test.wav",
    sampleRate: 48000,
    sampleLength: 480000,
    warping: true,
    warpMarkers: [
        { sampleTime: 0, beatTime: 0 },
        { sampleTime: 5, beatTime: 8 },
        { sampleTime: 10, beatTime: 20 }
    ]
};
```

The renderer and controllers should work from this data without Ableton running.

This allows coding agents without Live access to implement most of the project.

---

# 60. Manual Ableton test suite

Before every release manually verify:

```text
Session clip

Arrangement clip

warped clip

unwarped clip

looped clip

non-looped clip

clip with start offset

clip with many warp markers

short clip

long clip

mono clip

stereo clip

44.1 kHz

48 kHz

96 kHz

clip file moved/deleted

clip changed during analysis

Live transport start/stop

Live transport jump

loop boundary crossing

warp marker editing while device open
```

Record outcome in:

```text
docs/TESTING.md
```

---

# 61. File-format matrix

Before release test:

```text
WAV PCM16

WAV PCM24

WAV Float32

AIFF

MP3 320

MP3 VBR

FLAC

M4A/AAC

OGG
```

Do not assume one MP3 test covers all compressed audio cases.

---

# 62. Long-file testing

Test:

```text
5 minutes

15 minutes

60 minutes

2 hours
```

The system should remain responsive.

A long audio file must not make rendering slower merely because it contains more source frames.

That is why the pyramid architecture is required.

---

# 63. Memory testing

Instrument analyzer memory.

Log optionally in development:

```text
PCM memory

analysis array memory

pyramid memory

cache memory

peak heap
```

If decoder architecture requires loading the entire uncompressed stereo song into RAM, treat it as temporary MVP behavior.

Release architecture should preferably analyze streaming/chunked PCM.

---

# 64. Logging

Implement levels:

```text
ERROR

WARN

INFO

DEBUG
```

Production default:

```text
WARN
```

Developer mode:

```text
DEBUG
```

Example:

```text
[SpectraWave][Analyzer]
Analyzing /Music/Test.wav

[SpectraWave][Cache]
Hit 18ac39...

[SpectraWave][Warp]
Loaded 73 markers
```

Never spam Max's console every animation frame.

---

# 65. Configuration

Centralize defaults:

```typescript
const DEFAULT_CONFIG = {
    analysisRate: 150,

    fftSize: 2048,

    lowMaxHz: 250,

    midMaxHz: 4000,

    normalizationPercentile: 99.5,

    gamma: 0.65,

    targetFps: 30
};
```

Do not expose all of these to users initially.

Most are developer tuning parameters.

---

# 66. Visual calibration tool

Create an internal development mode with controls for:

```text
low crossover

high crossover

gamma

compression factor

normalization percentile

band opacity
```

Display the same test track while tuning.

Once good defaults are selected, hide advanced controls from normal UI.

---

# 67. Comparison corpus

Create a local test corpus containing music examples representing:

```text
techno

house

drum & bass

rock

acoustic

hip-hop

ambient

classical
```

The goal is not genre detection.

The purpose is ensuring the 3-band waveform communicates musical structure across different spectral balances.

Do not commit copyrighted commercial songs into the repository.

Use legally distributable test material or user-provided local tracks.

---

# 68. Reference implementation caution

WavScope is an existing open-source Max for Live waveform visualizer and may be useful for studying practical Max waveform techniques. Its repository is GPL-3.0 licensed. ([GitHub][10])

Therefore:

**Do not copy WavScope code into SpectraWave unless we intentionally choose GPL-compatible licensing for SpectraWave.**

It can be studied as a behavioral/reference project.

Record any borrowed architectural idea independently and implement it ourselves unless license compatibility is deliberately accepted.

---

# 69. Product identity

Do not market the product as:

```text
Rekordbox for Ableton
```

or imply AlphaTheta involvement.

Use language such as:

```text
3-band spectral waveform

DJ-style multiband waveform

frequency-colored waveform
```

Rekordbox is a design reference for the feature concept, not product branding.

---

# 70. Definition of MVP complete

The MVP is complete when all of the following are true:

```text
✓ Max for Live device loads normally

✓ audio passes through unchanged

✓ selecting an Ableton audio clip is detected

✓ file path is obtained automatically

✓ the source file is analyzed

✓ LOW/MID/HIGH waveform is displayed

✓ waveform analysis is cached

✓ repeat selection loads cache

✓ playhead follows playback

✓ warped clips remain synchronized

✓ unwarped clips remain synchronized

✓ zoom works

✓ follow mode works

✓ overview works

✓ loop region is shown

✓ no UI blocking during analysis

✓ stale analysis jobs cannot replace current clip

✓ common error states are handled

✓ core mathematics has automated tests
```

Only after this should the project be considered a functional product rather than a prototype.

---

# 71. Definition of V1 release complete

V1 requires everything in MVP plus:

```text
✓ Windows tested

✓ macOS tested

✓ WAV tested

✓ AIFF tested

✓ MP3 tested

✓ FLAC tested

✓ M4A tested

✓ OGG tested

✓ Session View tested

✓ Arrangement View tested

✓ Selected Clip mode

✓ Playing/Track mode

✓ persistent cache

✓ cache invalidation

✓ analysis cancellation

✓ stable 30 FPS rendering

✓ long-file test

✓ missing-file behavior

✓ corrupt-cache recovery

✓ packaged device documentation
```

---

# 72. Priority order

When faced with competing work, use this priority:

```text
1. Correct synchronization

2. Application stability

3. Audio transparency

4. Correct spectral analysis

5. Responsive UI

6. Cache behavior

7. Format support

8. Visual polish

9. Additional features
```

A beautiful waveform that is 100 ms out of sync is considered broken.

---

# 73. Agent rule: no speculative rewrites

Once a subsystem passes its acceptance tests, do not rewrite it solely for aesthetic code reasons.

Refactor only when:

```text
measurable performance issue

bug

clear architectural blocker

testability problem

dependency problem
```

This project crosses Max, Ableton APIs, Node and audio analysis. Unnecessary rewrites will introduce integration regressions.

---

# 74. First exact tasks for the coding agent

Start with the following execution sequence:

```text
TASK 001
Create repository structure.

TASK 002
Create docs/STATUS.md.

TASK 003
Create docs/HANDOFF.md.

TASK 004
Create docs/DECISIONS.md.

TASK 005
Create minimal transparent Max Audio Effect.

TASK 006
Connect to:
live_set view detail_clip

TASK 007
Print selected clip ID.

TASK 008
Read:
is_audio_clip
name
file_path
sample_rate
sample_length
warping

TASK 009
Read and serialize warp_markers.

TASK 010
Observe clip changes.

TASK 011
Observe playing_position.

TASK 012
Create synthetic v8ui 3-band waveform.

TASK 013
Benchmark waveform rendering.

TASK 014
Create Node for Max service.

TASK 015
Verify request/response protocol.

TASK 016
Implement WAV decoder.

TASK 017
Generate sine-wave test fixtures.

TASK 018
Implement STFT.

TASK 019
Implement 3-band reduction.

TASK 020
Create static waveform from selected WAV clip.

TASK 021
Implement cache.

TASK 022
Implement WarpMap unit tests.

TASK 023
Connect playhead to source time.

TASK 024
Test warped impulse track.

TASK 025
Begin scrolling viewport.
```

Do these in order unless a task is genuinely blocked.

---

# 75. First major milestone

The first milestone is called:

## M1 — Living Waveform

It should demonstrate:

```text
1. Open Ableton.

2. Add SpectraWave.

3. Click an audio clip.

4. SpectraWave detects it.

5. SpectraWave analyzes it.

6. Blue/orange/white waveform appears.

7. Press Play.

8. Playhead moves correctly.

9. Change warp markers.

10. SpectraWave updates geometry without re-analyzing audio.
```

This is the first moment where the core concept has been proven.

Do not prioritize additional features until M1 works reliably.

---

# 76. Second major milestone

## M2 — DJ View

Add:

```text
centered scrolling playhead

musical zoom

bar/beat grid

full-track overview

loop display

smooth playback
```

At this point the device should begin to feel like the desired product rather than an engineering prototype.

---

# 77. Third major milestone

## M3 — Daily Driver

Add:

```text
compressed audio support

Playing Clip mode

Arrangement support

robust caching

Windows/macOS validation

error recovery

performance optimization
```

This is the candidate for V1 release.

---

# 78. Longer-term architecture

Keep boundaries clean enough that a future native version could replace pieces individually:

```text
M4L LOM Adapter
        │
        ▼
Shared core formats
        │
 ┌──────┴──────┐
 ▼             ▼
Node           Native C++
Analyzer       Analyzer

        │
        ▼
waveform cache
        │
 ┌──────┴───────┐
 ▼              ▼
MGraphics       GPU renderer
```

The cache format and warp algorithms should therefore not depend directly on Max internals.

---

# 79. Final engineering principle

The project should treat these as three separate problems:

```text
WHAT AUDIO DOES THE CLIP CONTAIN?

          +

WHERE DOES THAT AUDIO OCCUR
ON ABLETON'S MUSICAL TIMELINE?

          +

HOW SHOULD THAT INFORMATION
BE DRAWN?
```

Those correspond to:

```text
ANALYZER

WARP MAP

RENDERER
```

Keep them independent.

That separation is the most important architectural decision in SpectraWave.

If implemented this way, later features such as spectral coloring, phrase detection, stems, enlarged views, native rendering or even other DAWs can be added without rebuilding the foundation.

---

# 80. Current API assumptions verified for this plan

The implementation plan is based on current Live/Max APIs in which:

* `Song.View.detail_clip` exposes the clip displayed in Live's Detail View. ([Cycling '74 Documentation][1])
* audio clips expose their underlying `file_path`. ([Cycling '74 Documentation][6])
* clips expose `sample_length` and `sample_rate`. ([Cycling '74 Documentation][6])
* clips expose `playing_position`. ([Cycling '74 Documentation][5])
* warped clips expose `warp_markers`, including `sample_time` and `beat_time`. ([Cycling '74 Documentation][5])
* tracks expose Session `clip_slots` and Arrangement `arrangement_clips`. ([Cycling '74 Documentation][9])
* `v8ui` provides modern JavaScript UI support with MGraphics. ([Cycling '74 Documentation][7])
* Node for Max can run a separate Node process and communicate with Max through `node.script`. ([Cycling '74 Documentation][2])

These capabilities are sufficient to implement the proposed architecture without modifying Ableton itself.

---

# 81. Instruction to the implementation agent

Build SpectraWave incrementally according to the phase gates in this document.

Prioritize working tested vertical slices over incomplete large-scale implementation.

At every stage:

```text
IMPLEMENT
   ↓
TEST
   ↓
PROFILE
   ↓
DOCUMENT
   ↓
COMMIT
   ↓
CONTINUE
```

Never report a feature as working unless it has actually been tested.

If the environment lacks Ableton or Max, continue implementing and testing all pure TypeScript/JavaScript portions using mocks and fixtures rather than stopping the project.

Clearly mark Live-dependent tests as:

```text
AWAITING LIVE VALIDATION
```

rather than pretending they passed.

Continue autonomously through all unblocked tasks and maintain `STATUS.md` and `HANDOFF.md` so that another agent can resume the project immediately.

One architectural change I consider especially important compared with our initial brainstorm is the **multi-resolution waveform pyramid**. That makes a two-hour file effectively no harder to draw than a five-minute file, and it gives us a strong foundation for very smooth zooming.

I would give the agent this document as `IMPLEMENTATION_PLAN.md` alongside your usual `PROJECT_BOOTSTRAP.md`; the first real target should be **M1 — Living Waveform**, before it spends time making the interface pretty.

[1]: https://docs.cycling74.com/apiref/lom/song_view/?utm_source=chatgpt.com "Song.View - Live Object Model | Cycling '74 Documentation"
[2]: https://docs.cycling74.com/reference/node.script?utm_source=chatgpt.com "node.script - Node for Max Reference | Cycling '74 Documentation"
[3]: https://docs.cycling74.com/reference/sfplay~/?utm_source=chatgpt.com "sfplay~ - MSP Reference | Cycling '74 Documentation"
[4]: https://docs.cycling74.com/reference/buffer~/?utm_source=chatgpt.com "buffer~ - MSP Reference | Cycling '74 Documentation"
[5]: https://docs.cycling74.com/apiref/lom/clip/?utm_source=chatgpt.com "Clip - Live Object Model | Cycling '74 Documentation"
[6]: https://docs.cycling74.com/apiref/lom/clip/ "Clip - Live Object Model | Cycling '74 Documentation"
[7]: https://docs.cycling74.com/reference/v8ui/?utm_source=chatgpt.com "v8ui - Max Reference | Cycling '74 Documentation"
[8]: https://docs.cycling74.com/apiref/js/mgraphics/?utm_source=chatgpt.com "MGraphics - Max JS API | Cycling '74 Documentation"
[9]: https://docs.cycling74.com/apiref/lom/track/?utm_source=chatgpt.com "Track - Live Object Model | Cycling '74 Documentation"
[10]: https://github.com/zsteinkamp/m4l-WavScope?utm_source=chatgpt.com "GitHub - zsteinkamp/m4l-WavScope: Waveform visualizer device for Ableton Live · GitHub"
