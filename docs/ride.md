# Ride Cymbal Generator Documentation

The `render_ride` generator is a specialized ride cymbal synthesis engine in Drumlabooh designed to model shimmering ride sustain, clean stick definition, and airy high-frequency wash. It features a high-frequency extended resonant filter section, an inharmonic 6-oscillator sine cluster, an Xorshift noise-to-metal balance control, an ultra-short glassy tip "ping" transient with high-frequency sparkle, a smooth release tail fade-out, and a soft-clipping saturation stage.

Call format:
`ride(decay=val1, pitch=val2, tone=val3, drive=val4, noise=val5, ping=val6)`

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`decay`** | `float` | `1.8` | `0.1` — `8.0` | Overall duration factor governing the envelope tail length (in seconds). |
| **`pitch`** | `float` | `1.1` | `0.3` — `3.0` | Global pitch tuning multiplier applied to the base metallic frequency. |
| **`tone`** | `float` | `0.6` | `0.0` — `1.0` | Brightness and filter cutoff frequency. Values $\le 1.0$ map automatically to $[4000.0, 14000.0]\text{ Hz}$. |
| **`drive`** | `float` | `1.1` | `1.0` — `5.0` | Soft-clipping saturation multiplier applied via `std::tanh`. |
| **`noise`** | `float` | `0.25` | `0.0` — `1.0` | Balance ratio between raw Xorshift noise wash and the inharmonic sine ring cluster. |
| **`ping`** | `float` | `0.6` | `0.0` — `2.0` | Intensity and sparkle level of the ultra-short glass-like stick-strike attack. |

---

## Architecture and Sonic Features

1. **Inharmonic Sine Cluster & Noise Balance:**
   - Six independent sine oscillators are driven by golden-ratio-derived inharmonic intervals based on `base_metal` ($1.000\times$, $1.618\times$, $2.542\times$, $3.833\times$, $5.211\times$, $7.145\times$):
     $$\text{metallic\_ring} = (\text{osc}_1 \cdot \text{osc}_3 + \text{osc}_2 \cdot \text{osc}_4 + \text{osc}_5 \cdot \text{osc}_6) \cdot 0.22$$
   - Combined with Xorshift noise according to the `noise` mix parameter (`combined = raw_noise * noise + metallic_ring * (1.0 - noise)`).

2. **Extended High-Frequency Resonant Filtering:**
   - The combined signal passes through a smooth biquad/state-variable style filter stage configured with variable frequency `w0` (clamped at $0.92$) and a gentle resonance factor $Q = 1.4$ to prevent harsh resonant ringing while retaining air and brilliance up to $14\text{ kHz}$.

3. **Ultra-Short Glassy "Ping" Attack:**
   - An extremely short transient stage ($0.8\text{ ms}$, $t < 0.0008\text{ s}$) injects a high-frequency needle-sharp stick sparkle (`stick_sparkle = std::sin(t * 18000.0 * M_PI) * 0.5`) scaled by `ping`.

4. **Tail Release & Soft Saturation:**
   - A built-in linear release fade (`release_fade`) operates across the final $20\%$ of the decay duration (`tail_time = decay * 0.2`) for a natural, click-free finish.
   - Overdrive saturation (`std::tanh`) adds harmonic density when `drive > 1.0`.
