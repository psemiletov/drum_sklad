# Metal / Cymbal / Percussion Generator Documentation

The `render_metal` generator is a specialized metallic percussion and cymbal synthesis engine in Drumlabooh. It models metallic resonance, bells, and industrial high-frequency percussive hits by combining an inharmonic cluster of six distinct oscillators, a pseudo-random noise source, a resonant band-pass filter stage, independent exponential envelope decays for body and noise components, a sharp attack click transient, and a built-in envelope-driven release stage with a soft-clipping saturation circuit.

Call format:
`metal(decay=val1, metal=val2, tone=val3, drive=val4)`

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`decay`** | `float` | `1.2` | `0.1` — `5.0` | Overall duration factor governing the envelope tail length (in seconds). |
| **`metal`** | `float` | `750.0` | `200.0` — `3000.0` | Base frequency governing the inharmonic metallic oscillator cluster (in Hz). |
| **`tone`** | `float` | `5500.0` | `1000.0` — `12000.0` | Brightness and cutoff frequency of the resonant band-pass filter (in Hz). |
| **`drive`** | `float` | `1.3` | `1.0` — `5.0` | Soft-clipping saturation multiplier applied via `std::tanh`. |

---

## Architecture and Sonic Features

1. **Inharmonic Metallic Cluster:**
   - Six independent phase accumulators are driven by frequency ratios derived from `metal` ($1.000\times$, $1.473\times$, $2.154\times$, $3.112\times$, $4.391\times$, $5.817\times$) to simulate bell-like inharmonic metal modes:
     $$\text{metallic\_ring} = (\text{osc}_1 \cdot \text{osc}_3 + \text{osc}_2 \cdot \text{osc}_4 + \text{osc}_5 \cdot \text{osc}_6) \cdot 0.33$$

2. **Noise-Metallic Mixing & Resonant Band-Pass Filtering:**
   - A pseudo-random noise generator (Xorshift) is blended with the metallic ring cluster ($35\%$ noise and $65\%$ metallic ring).
   - The combined signal passes through a 2-pole resonant band-pass filter (`bpf_s1`, `bpf_s2`) configured with variable frequency `w0` (clamped at $0.8$ for stability) and a resonance factor $Q = 2.0$.

3. **Multi-Stage Envelopes & Attack Click:**
   - **Body and Noise Envelopes:** Separate exponential decay rates shape the noise and metallic body, ensuring the noise decays faster than the metallic tail:
     $$\text{body\_env} = \exp\left(-t \cdot \frac{4.5}{\text{decay}}\right), \quad \text{noise\_env} = \exp\left(-t \cdot \frac{6.0}{\text{decay}}\right)$$
   - **Attack Transient:** A short linear ramp transient ($0$ to $2\text{ ms}$, $t < 0.002\text{ s}$) adds an initial metallic stick or striker click.

4. **Custom Release Tail & Soft Saturation:**
   - A built-in linear release fade is applied across the final $20\%$ of the decay duration (`tail_time = decay * 0.2`) to ensure a smooth, natural extinction without relying on external post-processing fade-outs.
   - If `drive > 1.0`, the final sample passes through a `std::tanh` saturation stage to add harmonic warmth and control peaks.
