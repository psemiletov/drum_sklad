# Goa Snare Generator Documentation

The `goasnare` generator is a vintage snare drum synthesis engine in Drumlabooh engineered to replicate the classic 1990s Goa Trance and Psytrance snare sound. It is modeled after iconic hardware setups of the era (such as the Roland TR-909 snare, Akai S1000 sampler, and overdriven Mackie/Soundcraft analog mixing consoles). It utilizes a dual-tone body engine, band-pass filtered white noise for snare wires, exponential envelope decays, non-linear console saturation drive, and optional lo-fi bit quantization.

Call format:
`goasnare(tone_freq=val1, tone_decay=val2, noise_freq=val3, noise_decay=val4, snappy=val5, drive=val6, bit_depth=val7)`

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`tone_freq`** | `float` | `180.0` | `130.0` — `300.0` | Base pitch frequency of the primary body oscillator (in Hz). |
| **`tone_decay`** | `float` | `0.08` | `0.03` — `0.20` | Exponential decay duration of the tonal body (in seconds). |
| **`noise_freq`** | `float` | `3200.0` | `1500.0` — `6000.0` | Center frequency of the 2-pole Band-Pass filter for noise (in Hz). |
| **`noise_decay`**| `float` | `0.16` | `0.05` — `0.30` | Exponential decay duration of the snare wire noise tail (in seconds). |
| **`snappy`** | `float` | `0.50` | `0.0` — `1.0` | Mix ratio balancing the tonal body (`0.0`) against the noise tail (`1.0`). |
| **`drive`** | `float` | `0.35` | `0.0` — `1.2` | Console preamp saturation intensity passed through `std::tanh`. |
| **`bit_depth`** | `float` | `16.0` | `12.0` — `16.0` | Quantization bit depth simulating 1990s sampler DAC grit (`12.0` = Akai S1000). |

---

## Architecture and Sonic Features

1. **Dual-Tone Body Engine:**
   Two sine wave oscillators simulate the acoustic shell resonance of a TR-909 style snare drum:
   - Primary Oscillator running at $f_1 = \text{tone\_freq} + 60 \cdot \exp(-40t)$.
   - Interval Oscillator running at $f_2 = \text{tone\_freq} \cdot 1.55 + 80 \cdot \exp(-40t)$.
   
   The fast pitch drop during the initial attack phase delivers a tight, wooden stick impact. The two tones are mixed in a $60\% / 40\%$ ratio and modulated by an exponential decay envelope.

2. **Band-Pass Filtered Snare Wires:**
   White noise generated via Xorshift PRNG passes through a resonant 2-pole Band-Pass filter (with $Q = 1.8$) centered around `noise_freq`. This isolates the snare wire rattle to specific mid/high frequency bands, preventing low-frequency clutter in the mix.

3. **Tone/Noise Balance & Saturation Drive:**
   The tonal shell and filtered noise signals are blended according to `snappy`:
   $$\text{sample} = \text{tone\_signal} \cdot (1.0 - 0.7 \cdot \text{snappy}) + \text{noise\_signal} \cdot (1.3 \cdot \text{snappy})$$
   When $\text{drive} > 0.01$, the mixed sum passes through an analog console preamp emulation stage:
   $$\text{sample} = \tanh\left(\text{sample} \cdot (1.0 + 2.5 \cdot \text{drive})\right)$$

4. **Lo-Fi DAC Quantization:**
   When $\text{bit\_depth} < 15.9$, the signal undergoes bit-depth reduction and amplitude quantization:
   $$\text{steps} = 2^{(\text{bit\_depth} - 1.0)}, \quad \text{sample} = \frac{\text{round}(\text{sample} \cdot \text{steps})}{\text{steps}}$$
   This accurately reproduces the lo-fi texture and harmonic crunch of 12-bit hardware samplers like the Akai S1000, followed by a $3\text{ ms}$ fade-out smoothing stage (`apply_buffer_fadeout`).
