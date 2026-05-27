


#  AMB - Analog Mastering Bundle v1.0  no GUI

**AMB - PE3  - 3 Band Parametric EQ
**AMB - PE5  - 5 Band Parametric EQ
**AMB - GE10 - 10 Bang Graphic EQ 
**AMB - BC   - Bus Compressor
**AMB - MBC  - Master Bus Compressor




---

## ✨ Shared features

- 🎚️ **State Variable Filter** (Shelf · Bell · Butterworth) with NaN protection
- 📻 **Analog emulation** — op-amp API 2520 model, gain-compensated
- 🎛️ **Drive** (0–100%) — continuous saturation, constant perceived loudness
- 🔁 **Oversampling (zero-latency)** — minimum-phase IIR half-band (all-pass)
  oversampler pre anti-aliasing saturačného stupňa, bez pre-ringingu a bez PDC.
  (Pozn.: faktor je v aktuálnej verzii fixne 32×; prepínač 16×/32× je pripravený.)
- ⚡ **Runtime SIMD dispatch** — AVX-512 → AVX2 → Scalar, **selected automatically**
  podľa CPU. Vektorizovaná je saturácia, ktorá beží na nadvzorkovanom bloku
  (16–32 vzoriek naraz) priamo v oversampleri — teda na najťažšej časti reťazca.
- 🔧 **Continuous Q** — smooth bandwidth control (narrow ↔ wide)
- 🎚️ **Gain ±24 dB** per band, smoothed (no zipper noise)
- 📻 **Bandpass** 80 Hz – 10 kHz (Butterworth HP + LP)
- 🔌 **CLAP / VST3**, stereo (independent L/R)



## 🏗️ Layout (Cargo workspace)

```
lap-amb/
├── Cargo.toml                 # workspace root (one build for everything)
├── xtask/                     # bundler (cargo xtask bundle ...)
├── crates/
│   ├── lap-dsp/              # shared DSP engine
│   │   └── src/
│   │       ├── filter.rs          # SVF filters (control/audio-rate split)
│   │       ├── filter_polyphase.rs# SVF (serial, používaný pluginmi)
│   │       ├── common.rs          # shared types (FilterShape, q_map, ...)
│   │       ├── oversampling.rs    # zero-latency IIR half-band oversampler
│   │       └── simd/              # runtime SIMD dispatch (saturácia)
│   │           ├── mod.rs         # CPU detection + saturate_block_auto()
│   │           ├── scalar.rs      # reference path (runs everywhere)
│   │           ├── avx2.rs        # 8 samples/cycle
│   │           └── avx512.rs      # 16 samples/cycle
│   ├── ambpe3/                # 3-band parametric EQ
│   ├── ambpe5/                # 5-band parametric EQ
│   ├── ambge10/               # 10-band graphic EQ
│   ├── ambbc/                 # Bus Compressor
│   └── ambmbc/                # Master Bus Compressor
```

