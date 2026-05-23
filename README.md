# ApEQ Series — Parametric EQ for Linux Audio

ApEQ Series is a line of professional parametric equalizers built in
Rust on top of the nih-plug framework. Two editions, one shared
DSP engine, **one build** with automatic SIMD selection based on the CPU.

   LAP  Linux Audio Plugin CLAP (Linux)

 Two editions — the name equals the band count

| Edition | Bands | Layout | Freqs/band | Character |
|---------|:-----:|--------|:----------:|-----------|
| lap-ApEQ-3 | 3 | Low · Mid · High | 5 | Classic, wider spacing |
| lap-ApEQ-4 | 4 | Low · Low-Mid · High-Mid · High | 7 | Surgical, denser spacing |


Shared features no GUI

*State Variable Filter (Shelf · Bell · Butterworth) with NaN protection
*Analog emulation — op-amp API 2520 model, gain-compensated
*Drive (0–100%) — continuous saturation, constant perceived loudness
*Oversampling 16× / 32× — polyphase FIR, anti-aliasing for the saturation
*stage, with a short crossfade on mode change to avoid clicks
*Runtime SIMD dispatch — AVX-512 → AVX2 → Scalar, selected automatically
*Continuous Q — smooth bandwidth control (narrow ↔ wide)
*Gain ±24 dB per band, smoothed (no zipper noise)
*Bandpass 80 Hz – 10 kHz (Butterworth HP + LP)
*CLAP , stereo (independent L/R)

*Note on oversampling: it is anti-aliasing, not an audible "effect". Its
*job is to keep the saturation clean at high drive — the better it works, the
**less* you hear it. The difference is most measurable at high drive on
*harmonically rich material (e.g. saw leads).


Layout (Cargo workspace)

```
lap-plugin/
├── Cargo.toml                 # workspace root (one build for everything)
├── crates/
│   ├── apeq-dsp/              # shared DSP engine
│   │   └── src/
│   │       ├── filter.rs          # SVF filters (control/audio-rate split)
│   │       ├── common.rs          # shared types (FilterShape, q_map, ...)
│   │       ├── oversampling.rs    # polyphase FIR oversampler (16× / 32×)
│   │       ├── fir_coeffs.rs      # auto-generated FIR coefficients
│   │       └── simd/              # runtime SIMD dispatch
│   │           ├── mod.rs         # CPU detection + saturate_block_auto()
│   │           ├── scalar.rs      # reference path (runs everywhere)
│   │           ├── avx2.rs        # 8 samples/cycle
│   │           └── avx512.rs      # 16 samples/cycle
│   ├── apeq3/                 # 3-band plugin (lib.rs, params.rs, processor.rs)
│   └── apeq4/                 # 4-band plugin (lib.rs, params.rs, processor.rs)
└── docs/                      # manual, installation, development notes
```

That's it. This produces two libraries (ApEQ3 + ApEQ4). No separate
AVX2/AVX-512 builds are required — the plugin detects the CPU's capabilities
once at startup and from then on uses the fastest available path. The same
binary runs on old and new processors alike.

| Platform | ApEQ3 | ApEQ4 |
|----------|-------|-------|
| Linux |

Then in your DAW: rescan plugins and look for "ApEQ3" or "ApEQ4".

Parameters

Per band:
Frequency — 5 steps (ApEQ3) or 7 steps (ApEQ4) per band
Gain — ±24 dB, smoothed
Shape — Shelf or Bell (Low / High bands; Mid is always Bell)

Global:
Q — continuous bandwidth (narrow ↔ wide), shared across bands
Drive — 0–100% op-amp saturation (gain-compensated; 0% = clean bypass)
Oversampling — Off / 16× / 32× (applies to the saturation stage)
Bandpass — on/off, 80 Hz – 10 kHz
Output — output gain


Status — v1.0 beta

This is a **beta**. The DSP core is complete and verified, but it has been
tested primarily on Linux (Bitwig). Bug reports and feedback are very welcome —
that's exactly why it's here. If something sounds wrong or won't build on your
setup, please open an issue.

Known notes:
- The op-amp gain compensation holds loudness within roughly ±1 dB across the
  drive range; at extreme drive it may drift slightly (physics of hard
  saturation).
- Oversampling adds a small latency (not yet reported to the host).

