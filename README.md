# AMB — Analog Mastering Bundle

A suite of mastering-focused audio plugins (CLAP + VST3) built in Rust on
[`nih_plug`](https://github.com/robbert-vdh/nih-plug). One workspace, one
build for everything, shared DSP engine, automatic SIMD dispatch.

## 🎛️ Plugins no GUI

| Plugin       | Description                                     | Gain range   |
|--------------|-------------------------------------------------|--------------|
| **AMB-PE3**  | 3-Band Parametric EQ                            | ±10 dB / 2 dB steps |
| **AMB-PE5**  | 5-Band Parametric EQ                            | ±12 dB / 2 dB steps |
| **AMB-PE8**  | 8-Band Mastering Parametric EQ *(new in v2.7)*  | ±15 dB / 3 dB steps |
| **AMB-GE10** | 10-Band Graphic EQ (ISO bands)                  | continuous   |
| **AMB-BC**   | Bus Compressor (with external sidechain)        | —            |
| **AMB-MBC**  | Master Bus Compressor (SSL-style)               | —            |


---

## ✨ Shared design language

All AMB plugins share the same DSP foundation and the same analog-character
philosophy. **This is not a placeholder — it's a deliberate design.**

### Always-on idle analog character

When you load any AMB plugin, the signal is **never bit-perfect identical
to the input**, even at "neutral" settings. The chain mirrors a real analog
console — wires, opamps and capacitors are in the signal path whether you
push them or not. Three components are intentionally fixed and always-on:

- 🌊 **Background noise floor** (`-85 dBFS` for EQ/BC, `-95 dBFS` for MBC) —
  idle console hiss. **Always running**, regardless of the `Analog On/Off`
  toggle. No user control.
- ⚡ **32× IIR oversampler** for the saturation stage. Factor is **fixed**;
  no user choice. Zero-latency (minimum-phase IIR half-band cascade).
- 🎚️ **Drive constant** (`0.4` for PE3/PE5/PE8, `0.25` for GE10/BC/MBC) —
  saturation amount when `Analog` is on. **Fixed**, calibrated per plugin.
  No additional saturation knob.

The only user-facing switch for this whole stage is the single **Analog On/Off**
toggle per plugin. When `Analog Off`, drive collapses to 0 (saturation is
skipped — cheap), but the background noise still runs. When `Analog On`,
saturation engages through the 32× oversampler and the drive gain compensates.

**Why this matters when you read the code:** in `processor.rs` you'll see
explicit `[NOISE]` / `[DRIVE]` / `[SAT]` / `[OS]` markers and a `§ DESIGN
CONSTANTS — DO NOT TREAT AS "DEAD" CODE` block at the top of every file.
Those tags are there so future edits don't accidentally re-introduce the
`if analog_enabled { noise }` gate that was removed in v2.7.

### Filter & EQ engine

- 🎚️ **State Variable Filter** (Cytomic / Simper-style; one canonical
  `SvfFilter` type since v2.2). Shelf · Bell · Lowpass · Highpass · Butterworth.
  NaN protection built in.
- 🔁 **HF anti-cramping** — every EQ band runs inside a 2× IIR half-band
  oversampler (`EqOversampler2x`). Pushes Nyquist out of the audible range,
  drops HF cramping error from ~2.5 dB to ~0.7 dB on a +12 dB @ 8 kHz bell.
- 🎛️ **Proportional-Q** *(PE3/PE5/PE8)* — Q widens with low gain (musical)
  and tightens with high gain (surgical). Curve: `Q(g) = 0.5 + 3.0 ×
  (|g|/max_gain)^0.75`. Normalized per plugin so all three hit Q=3.5 at full
  gain. Shelf Q is fixed at 0.7 (analog shelves don't change Q with gain).
- 🔘 **Stepped frequencies** — every EQ band is a stepped selector (Pultec /
  API style), not a continuous knob. Decisive workflow, repeatable settings.
- 🔇 **Click-free band Off** — when a band's shape is `Off`, its filter is
  set to `COEFFS_TRANSPARENT` (allpass), so toggling Off ↔ On is smooth.

### DC + Limiting

- ⚙️ **Sample-rate-correct DC blocker** — corner pinned to ~5 Hz at any
  sample rate (44.1k–192k). Previously the corner drifted to ~15 Hz @ 192k.
- 🛡️ **Soft limiter on MBC output** — transparent below ~-0.9 dBFS, smooth
  tanh ceiling at 0 dBFS. Replaces hard `clamp(-1, 1)` which was creating
  alias distortion at high makeup gain.

### SIMD

- ⚡ **Runtime SIMD dispatch** — AVX-512 → AVX2+FMA → scalar, picked
  automatically at first run. The saturation kernel (`Padé tanh` + asymmetric
  warmth) is vectorized inside the oversampler, so 16–32 samples are
  processed per CPU cycle on AVX-512 hardware. About 2× speedup measured vs
  scalar reference.

---

## 📋 EQ specifications

### PE3 (3 bands)

| Band  | Frequencies                | Shape options       |
|-------|----------------------------|---------------------|
| LOW   | 30 / 50 / 80 / 120 Hz      | Off · Low Shelf     |
| MID   | 1 / 2 / 3 / 5 kHz          | Off · Bell          |
| HIGH  | 10 / 12 / 15 / 20 kHz      | Off · High Shelf    |

### PE5 (5 bands, overlapping ranges)

| Band     | Frequencies                              | Shape options       |
|----------|------------------------------------------|---------------------|
| SUB      | 25 / 30 / 40 / 50 / 60 / 80 Hz           | Off · Low Shelf     |
| LOW      | 100 / 120 / 180 / 250 / 350 / 500 Hz     | Off · Bell          |
| MID      | 1 / 1.5 / 2 / 3 / 5 / 7 kHz              | Off · Bell          |
| MID HIGH | 3 / 5 / 7 / 10 / 12 / 15 kHz             | Off · Bell          |
| HIGH     | 10 / 12 / 15 / 18 / 20 / 22 kHz          | Off · High Shelf    |

### PE8 (8 bands, mastering)

| Band       | Frequencies                                            | Shape options    |
|------------|--------------------------------------------------------|------------------|
| SUB        | 20 / 25 / 30 / 40 / 50 / 60 / 80 / 100 Hz              | Off · Low Shelf  |
| LOW        | 50 / 60 / 80 / 100 / 120 / 150 / 180 / 250 Hz          | Off · Bell       |
| MID LOW    | 250 / 350 / 500 / 700 / 1k / 1.2k / 1.5k / 2k Hz       | Off · Bell       |
| MID        | 1.5 / 2 / 2.5 / 3 / 4 / 5 / 7 / 8 kHz                  | Off · Bell       |
| MID HIGH   | 2.5 / 3 / 5 / 7 / 8 / 10 / 12 / 15 kHz                 | Off · Bell       |
| HIGH       | 10 / 12 / 15 / 18 / 20 / 22 / 25 / 28 kHz              | Off · High Shelf |
| AIR        | 12 / 15 / 18 / 20 / 22 / 25 / 28 / 32 kHz              | Off · High Shelf |
| AIR ULTRA  | 15 / 18 / 20 / 22 / 25 / 28 / 32 / 40 kHz              | Off · High Shelf |

**Note on AIR / AIR ULTRA:** frequencies above 22 kHz only fully apply at
higher sample rates. Internally PE8 clamps to ~0.49× oversampled Nyquist
(= ~0.98× Fs), so at Fs 44.1/48 kHz the top steps cap at ~43/47 kHz; at
Fs 88.2/96 kHz they work as labeled.

---

## 🏗️ Workspace layout

```
lap-amb/
├── Cargo.toml                # workspace root (one build for everything)
├── rust-toolchain.toml       # Rust 1.89 (AVX-512 stable) + Linux/Windows targets
├── bundler.toml              # plugin bundle metadata
├── build.sh                  # Linux build script
├── build-windows.sh          # Windows cross-compile script
├── xtask/                    # cargo xtask bundle ... (CLAP/VST3 packaging)
└── crates/
    ├── lap-dsp/              # shared DSP engine
    │   └── src/
    │       ├── filter.rs            # canonical SvfFilter (Shelf · Bell · LP · HP)
    │       ├── common.rs            # FilterShape, q_map (incl. proportional_q)
    │       ├── dc.rs                # sample-rate-correct DcBlocker
    │       ├── limiter.rs           # soft_limit (transparent-knee)
    │       ├── oversampling.rs      # Oversampler (32× sat) + EqOversampler2x
    │       └── simd/
    │           ├── mod.rs           # CPU detect + saturate_block_auto()
    │           ├── scalar.rs        # reference path
    │           ├── avx2.rs          # 8 samples/cycle (AVX2 + FMA)
    │           └── avx512.rs        # 16 samples/cycle (AVX-512F)
    ├── ambpe3/               # 3-band parametric EQ
    ├── ambpe5/               # 5-band parametric EQ
    ├── ambpe8/               # 8-band mastering EQ  (new in v2.7)
    ├── ambge10/              # 10-band graphic EQ
    ├── ambbc/                # Bus compressor
    └── ambmbc/               # Master bus compressor (SSL-style)
```

---

## 🔧 Build

Requires **Rust ≥ 1.89** (AVX-512 intrinsics stabilized + `edition2024` in
transitive deps). The version is pinned in `rust-toolchain.toml`, so
`rustup` will install it automatically on first build.

### Linux

```bash
# one command builds everything (SIMD is selected at runtime)
./build.sh
# or directly:
cargo build --release

# bundle each plugin to .clap / .vst3:
cargo xtask bundle ambpe3 --release
cargo xtask bundle ambpe5 --release
cargo xtask bundle ambpe8 --release
cargo xtask bundle ambge10 --release
cargo xtask bundle ambbc --release
cargo xtask bundle ambmbc --release

# install (CLAP)
cp target/release/libambpe3.so  ~/.clap/AmbPE3.clap
cp target/release/libambpe5.so  ~/.clap/AmbPE5.clap
cp target/release/libambpe8.so  ~/.clap/AmbPE8.clap
cp target/release/libambge10.so ~/.clap/AmbGE10.clap
cp target/release/libambbc.so   ~/.clap/AmbBC.clap
cp target/release/libambmbc.so  ~/.clap/AmbMBC.clap
```

### Windows (cross-compile from Linux)

Prerequisites:

```bash
# 1. Install mingw-w64 toolchain
sudo apt install mingw-w64                  # Debian/Ubuntu
sudo dnf install mingw64-gcc                # Fedora

# 2. Add Rust target (already in rust-toolchain.toml; rustup will fetch it)
rustup target add x86_64-pc-windows-gnu
```

Build:

```bash
./build-windows.sh
# or directly:
cargo build --release --target x86_64-pc-windows-gnu

# bundle each plugin for Windows:
cargo xtask bundle ambpe3 --release --target x86_64-pc-windows-gnu
# (analogically for ambpe5, ambpe8, ambge10, ambbc, ambmbc)
```

Output `.dll` files land in `target/x86_64-pc-windows-gnu/release/`.

### Windows (native MSVC build)

Native Windows build works the same — install Rust 1.89 + MSVC build tools,
then `cargo build --release`. CLAP/VST3 bundling via `cargo xtask` works
the same as on Linux.

---

## 📜 v2.7 changes (current release)

### New plugin

- **AMB-PE8** — 8-band mastering parametric EQ (SUB, LOW, MID LOW, MID,
  MID HIGH, HIGH, AIR, AIR ULTRA). Stepped ±15 dB gain in 3 dB steps,
  Fs-aware top bands that scale up to 40 kHz at high sample rates.

### EQ overhaul (PE3, PE5, PE8)

- **Stepped frequencies and gains** — every EQ band selector is now stepped
  (Pultec / API style). PE3: 4 freqs × 3 bands, ±10 dB / 2 dB. PE5: 6 ×
  5, ±12 dB / 2 dB. PE8: 8 × 8, ±15 dB / 3 dB.
- **Proportional-Q** — new `lap_dsp::q_map::proportional_q(gain, max)`.
  All bell bands use it. Shared engine across PE3/PE5/PE8, normalized so
  Q=3.5 at full gain in any plugin.
- **Per-band shape enums** — every band has its own `Off / <shape>` enum
  (e.g. `LowShape::Off / LowShape::LowShelf`), so the param schema is
  precise per band instead of one generic `FilterShape`.
- **Click-free band Off** — `Off` state sets `SvfFilter::COEFFS_TRANSPARENT`
  (allpass) so the filter stays in the chain at unity, smooth Off ↔ On
  switching.

### Analog character — "always on" model

The `if analog_enabled` guard around the background noise generator was
**removed in all five plugins** (PE3, PE5, GE10, BC, MBC; PE8 ships with
it already removed). Noise now runs continuously at -85 dBFS (-95 dBFS in
MBC), modelling the idle floor of a real analog console.

The `Analog On/Off` toggle continues to control saturation + drive gain
only, exactly as before. **Saturation always routes through the 32× IIR
oversampler when Analog is On — no user choice on the oversampling factor.**

Every `processor.rs` now has prominent `[NOISE]` / `[DRIVE]` / `[SAT]` /
`[OS]` markers and a `§ DESIGN CONSTANTS` header block, explicitly saying
these are not dead code — to prevent future edits from accidentally
adding back the removed guards.

### MBC threshold knob

The MBC `Threshold` knob now ranges **-15 ... +15 dB centered at 0**
(displayed as dBu). Internally converted to dBFS via `DBU_TO_DBFS = -12`
(0 dBu = -12 dBFS, standard studio reference), preserving the original
detector calibration.

### Soft limiter on MBC

The hard `clamp(-1, 1)` on the MBC output (which was creating audible alias
distortion at high makeup gain) was replaced with `lap_dsp::soft_limit`:
transparent below ~-0.9 dBFS, smooth tanh ceiling at 0 dBFS. Always active.

### HF cramping fix on EQ

Every EQ band now runs inside a 2× IIR zero-latency oversampler
(`EqOversampler2x`). This pushes the effective Nyquist out, reducing
high-shelf / high-bell cramping error from ~2.5 dB to ~0.7 dB on a
+12 dB @ 8 kHz bell at 44.1 kHz (verified numerically vs 16× reference).
No latency added; no CPU concern (saturation 32× is still the dominant load).

### Sample-rate-correct DC blocker

The DC blocker corner is now pinned to ~5 Hz at any sample rate. The
previous fixed-pole implementation drifted from ~3.5 Hz @ 44.1k to ~15 Hz
@ 192k. New: `R = exp(-2π · 5 / Fs)`.

### Removed dead code

- Old `PolyphaseFilter` was merged into the canonical `SvfFilter` (it was
  identical after the v2.2 fix, just two type names). A `#[deprecated] type
  PolyphaseFilter = SvfFilter` alias remains for backwards compatibility.
- The `OversamplingMode::Off` branch and `xfade` crossfade machinery in EQ
  plugins were removed — they were unreachable code (`os_mode` is always
  `Mode32x`).

### SIMD reactivated

The pre-v2.2 code path had the SIMD saturation kernel registered but not
called (only the scalar `Off` branch was reached). Saturation now actually
runs vectorized through `Oversampler::process_saturate` → `saturate_block_auto`,
delivering the ~2× speedup the AVX-512 / AVX2 kernels were designed for.

### Real-time safety

All audio-thread allocations (`Vec::to_vec()` on sidechain copy and mono
duplication) were eliminated. Buffers are preallocated in `initialize()`
and reused via `copy_from_slice`. No `malloc` on the audio thread.

### Build system

- Added `ambpe8` to `bundler.toml` and workspace members.
- `rust-toolchain.toml` adds `x86_64-pc-windows-gnu` target so `rustup`
  fetches it automatically.
- New `build-windows.sh` cross-compile script.
- `build.sh` updated to include PE8.

---

## 📜 Version history

- **v2.7 (current)** — PE8, stepped freqs/gains, proportional-Q, "always on"
  noise, in-code design-constant markers, Windows cross-compile, MBC threshold
  knob centered at 0, README rewrite.
- **v2.6** — HF cramping fix via 2× EQ oversampler.
- **v2.5** — RT-safety cleanup: preallocated sidechain + mono buffers.
- **v2.4** — Soft limiter on MBC; removed dead `OversamplingMode::Off`.
- **v2.3** — Sample-rate-correct DC blocker.
- **v2.2** — `SvfFilter` / `PolyphaseFilter` merged into one canonical type.
- **v2.1** — Polyphase serial-state filter bug fixed (~8 dB error).
- **v2.0** — Initial workspace consolidation.

---

**License:** MIT · **Vendor:** LAP (Linux Audio Plugin) · **Bundle:** AMB
