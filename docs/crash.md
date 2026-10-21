# Crash Cymbal Generator Documentation

The `render_crash` generator is a specialized acoustic crash cymbal and splash percussion synthesis engine in Drumlabooh. It simulates realistic bronze cymbal physics by incorporating physical scaling based on cymbal size (in inches), an inharmonic 6-oscillator square wave cluster with customizable detuning, an Xorshift pseudo-random noise air-wash tail, a 1-pole high-pass filter (HPF), an initial stick-strike transient envelope, and a soft-clipping saturation stage.

Call format:
`crash(size=val1, decay=val2, tone=val3, noise=val4, metal=val5, pitch=val6, detune=val7, drive=val8)`

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`size`** | `float` | `18.0` | `12.0` — `24.0` | Physical cymbal diameter size in inches (rock standard is $18''$). Automatically scales pitch, filter tone, and tail decay if unset. |
| **`decay`** | `float` | `0.0` | `0.1` — `5.0` | Tail duration in seconds. If set to `0.0` (default), it automatically calculates based on `size` ($14'' \approx 1.2\text{s}$, $18'' \approx 1.9\text{s}$, $22'' \approx 2.6\text{s}$). |
| **`tone`** | `float` | `5500.0` | `1000.0` — `12000.0` | Base cutoff frequency of the high-pass filter shaping bronze brightness (in Hz). |
| **`noise`** | `float` | `0.6` | `0.0` — `1.0` | Mix proportion of the airy white noise wash tail. |
| **`metal`** | `float` | `0.4` | `0.0` — `1.0` | Mix proportion of the inharmonic square wave metallic harmonics. |
| **`pitch`** | `float` | `1.0` | `0.5` — `2.0` | Global pitch multiplier governing overall tuning. |
| **`detune`** | `float` | `0.35` | `0.0` — `1.0` | Metallic frequency spread/dispersion factor across the harmonic cluster. |
| **`drive`** | `float` | `1.2` | `1.0` — `5.0` | Aggressiveness and harmonic density multiplier via `std::tanh` saturation. |

---

## Architecture and Sonic Features

1. **Size-Based Physical Scaling:**
   - **Pitch Scaling:** Larger cymbals have a lower natural frequency. A size scale factor is derived relative to a $16''$ standard: $\text{size\_scale} = \frac{16.0}{\text{size}}$.
   - **Tone Scaling:** Larger cymbals inherently sound darker, adjusting the filter cutoff downwards: $\text{tone} \cdot (1.0 - (\text{size} - 14.0) \cdot 0.03)$.
   - **Decay Scaling:** Automatically extends tail duration for larger diameters when `decay <= 0.0`.

2. **Inharmonic Metallic Cluster & Noise Wash:**
   - Six square-wave oscillators (`base_metal_freqs`) with staggered detuning ratios simulate complex bronze plate modes:
     $$\text{metal\_sample} = \frac{1}{6} \sum_{m=0}^{5} \text{sq}(\text{phase}_m)$$
   - Combined with deterministic Xorshift noise according to `noise_mix` and `metal_mix` proportions.

3. **High-Pass Filtering & Transient Attack:**
   - The mixed raw signal passes through a 1-pole high-pass filter (HPF) to isolate the high-frequency shimmering bronze spectrum.
   - The envelope (`env`) provides a sharp stick-strike attack ramp during the first $3\text{ ms}$ ($t < 0.003\text{ s}$) followed by an exponential decay tail:
     $$\text{env} = \exp\left(-2.5 \cdot \frac{t}{\text{decay}}\right)$$

4. **Saturation and Fade-Out:**
   - Overdrive saturation (`std::tanh`) adds grit and punch when `drive > 1.0`.
   - A final $15\text{ ms}$ smoothing stage (`apply_buffer_fadeout`) cleans up the buffer tail.
