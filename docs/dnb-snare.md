# DnB Snare Generator Documentation

The `render_dnb_snare` generator is a specialized percussion synthesis engine in Drumlabooh built specifically for Drum and Bass, jungle, and high-energy electronic music. It models a sharp, driving acoustic-style snare drum by combining a fast-pitch-dropping tonal body core, an RC low-pass filtered noise generator simulating the snare wires (seточки), an initial transient attack accent, and a non-linear saturation stage to deliver punchy compression and density.

Call format:
`dnb_snare(body_freq=val1, decay=val2, snap=val3, tone=val4, drive=val5)`

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`body_freq`** | `float` | `210.0` | `100.0` — `400.0` | Base frequency of the tonal drum body shell (in Hz). |
| **`decay`** | `float` | `0.25` | `0.05` — `1.0` | Overall envelope decay duration governing tail length (in seconds). |
| **`snap`** | `float` | `2.0` | `0.1` — `5.0` | Intensity level multiplier of the snare wires (seточки) noise tail. |
| **`tone`** | `float` | `3500.0` | `1000.0` — `8000.0` | Cutoff frequency of the 1-pole RC low-pass filter shaping the noise (in Hz). |
| **`drive`** | `float` | `1.8` | `1.0` — `5.0` | Preamp compression and saturation density multiplier passed through `std::tanh`. |

---

## Architecture and Sonic Features

1. **Tonal Body Shell & Fast Pitch Drop:**
   - The primary body oscillator features an aggressive initial frequency drop for a tight, punchy shell impact:
     $$\text{cur\_freq} = \text{body\_freq} + \text{body\_freq} \cdot 0.5 \cdot \exp(-t \cdot 50.0)$$
   - The tonal shell signal is generated via phase accumulation and shaped by an exponential decay envelope:
     $$\text{body} = \sin(\text{phase}) \cdot \exp\left(-t \cdot \frac{6.0}{\text{decay}}\right) \cdot 0.6$$

2. **Filtered Snare Wires & Transient Accent:**
   - Raw white noise generated via Xorshift PRNG passes through a single-pole RC low-pass filter governed by `tone`:
     $$\text{lpf\_state} = \text{lpf\_state} + \alpha \cdot (\text{raw\_noise} - \text{lpf\_state})$$
   - The wire envelope (`wire_env`) combines a decay tail scaled by `snap` with a $2.5\times$ initial transient boost during the first $2\text{ ms}$ ($t < 0.002\text{ s}$) to emphasize stick crack.

3. **Saturation Compression & Fade-Out:**
   - The mixed shell and wire signals pass through a non-linear `std::tanh` saturation stage scaled by `drive` to achieve heavy compression and harmonic density.
   - A final $4\text{ ms}$ smoothing fade-out stage (`apply_buffer_fadeout`) finishes the buffer cleanly.
