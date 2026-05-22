# ApEQ Series 
Analog Professional EQ

## Overview

ApEQ is a professional 5-band parametric equalizer with analog modeling,
based on the legendary API hardware units.

### Features

- **5-Band Parametric EQ:** Low, Low-Mid, Mid, High-Mid, High
- **Dual Filter Types:** Shelf (Low/High) and Bell (Mid bands)
- **Analog Emulation:** API 2520 op-amp modeling
- **Frequency-Specific Tuning:** Optimized band placement for vocals, instruments
- **Real-time Parameter Automation:** Sample-accurate control
- **SIMD Optimization:** AVX512/AVX2 support (4-8× faster)
- **Zero Aliasing:** 16× internal oversampling on nonlinear processing
- **VST3 & CLAP:** Full plugin compatibility

## Band Controls

### Low Band (Sub-Bass)
- **Gain Range:** -12dB to +12dB
- **Frequencies:** 30, 50, 75, 100, 150 Hz
- **Default:** 50 Hz, Shelf type
- **Use Case:** Bass body, sub-low control

### Low-Mid Band (Warmth)
- **Gain Range:** -12dB to +12dB
- **Frequencies:** 100, 150, 200, 300, 450 Hz
- **Type:** Bell (peak/cut)
- **Use Case:** Vocal warmth, instrument body

### Mid Band (Presence)
- **Gain Range:** -12dB to +12dB
- **Frequencies:** 600Hz, 900Hz, 1.2kHz, 1.8kHz, 2.7kHz
- **Type:** Bell (peak/cut)
- **Use Case:** Vocal clarity, instrument presence

### High-Mid Band (Brilliance)
- **Gain Range:** -12dB to +12dB
- **Frequencies:** 2, 3, 4, 6, 8 kHz
- **Type:** Bell (peak/cut)
- **Use Case:** Detail, harmonic enhancement

### High Band (Air/Shimmer)
- **Gain Range:** -12dB to +12dB
- **Frequencies:** 5, 7, 10, 14, 18 kHz
- **Default:** 10 kHz, Shelf type
- **Use Case:** Brightness, air, sizzle

## Advanced Controls

### Output Gain
- **Range:** -18dB to +18dB
- **Purpose:** Makeup gain after EQ processing
- **Typical:** 0dB (no change)

### Bandpass Filter
- **Range:** 50Hz - 15kHz
- **Type:** Butterworth (Q=0.707)
- **Purpose:** Remove sub/ultra-high content
- **When to use:** Spoken word, reducing tape hiss

### Analog Emulation
- **On/Off Toggle**
- **Effect:** Adds harmonic saturation (API 2520 curve)
- **When to use:** Warmth, vintage character
- **CPU Cost:** +5-10% (depending on SIMD)

### Phase Control
- **Range:** 0° - 180°
- **Purpose:** Inverted polarity for phase alignment
- **Typical:** 0° (normal)

## Tips & Tricks

### Vocal Enhancement
1. Low Band: +2-4dB @ 100Hz (bottom)
2. Low-Mid: -1-2dB @ 200Hz (reduce muddiness)
3. Mid: +3-6dB @ 1.2kHz (presence)
4. High-Mid: +2-4dB @ 4kHz (clarity)
5. High: +1-2dB @ 10kHz (air)

### Bass Control
1. Low: +4-8dB @ 50Hz (body)
2. Low-Mid: +2dB @ 100Hz (definition)
3. Rest: Keep neutral or slight cuts

### Mastering
1. Low: 0dB or slight cut @ 50Hz (remove rumble)
2. Low-Mid: -1dB @ 200Hz (reduce muddiness)
3. Mid: Neutral or +1dB @ 1.2kHz
4. High-Mid: +2-3dB @ 4kHz (clarity)
5. High: +1-2dB @ 10kHz (air, but not too much)

## Technical Specifications

- **Frequency Response:** 20Hz - 20kHz (±0.5dB)
- **THD:** <0.1% @ 0dBFS (without analog emulation)
- **Filter Type:** SVF (State Variable Filter)
- **Latency:** 0 samples (plugin delay)
- **Buffer Size:** 64 - 16384 samples supported
- **CPU Usage:** 
  - Scalar: ~70% per voice
  - AVX2: ~20% per voice
  - AVX512: ~12% per voice

**Version:** 0.4.7 
 release v1.0 
