# Hi-Hat Generator Documentation

The `hihat` generator (supporting both `hihat_close` and `hihat_open` call aliases) is a specialized synth engine designed to synthesize realistic analog-style acoustic and electronic hi-hat sounds. It combines a 6-oscillator square wave metallic core, deterministic pseudo-random white noise, a high-pass filter (HPF), an exponential envelope generator, and dynamic saturation.

Call format:
```text
hihat(decay=val1, tone=val2, noise=val3, metal=val4, pitch=val5, detune=val6, drive=val7)
hihat_close(decay=val1, tone=val2, noise=val3, metal=val4, pitch=val5, detune=val6, drive=val7)
hihat_open(decay=val1, tone=val2, noise=val3, metal=val4, pitch=val5, detune=val6, drive=val7)
```

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`decay`** | `float` | `0.08` (Closed)<br>`0.35` (Open) | $> 0.0$ | Amplitude envelope decay time in seconds. Determines tail length ($1.2 \times \text{decay}$ total render buffer length). |
| **`tone`** | `float` | `7000.0` | $> 0.0$ | High-pass filter cutoff frequency in Hz. Higher values remove more low-mid frequencies for a crisp tone. |
| **`noise`** | `float` | `0.5` | `0.0` — `1.0` | Gain multiplier for deterministic pseudo-random white noise generator. |
| **`metal`** | `float` | `0.5` | `0.0` — `1.0` | Gain multiplier for the 6-oscillator square wave metallic generator bank. |
| **`pitch`** | `float` | `1.0` | $> 0.0$ | Master frequency scaling factor applied across all metallic square wave oscillators. |
| **`detune`** | `float` | `0.0` | $\ge 0.0$ | Inharmonic frequency dispersion factor across metallic oscillators ($1 + m \times \text{detune} \times 0.15$). |
| **`drive`** | `float` | `1.0` | $\ge 1.0$ | Saturation intensity parameter. Values $> 1.0$ apply hyperbolic tangent (`std::tanh`) wave-shaping. |

---

## Architecture and Sonic Features

1. **6-Oscillator Inharmonic Metallic Generator:**
   Generates a metallic timbre using 6 square wave oscillators ($f_m \in \{405.3, 535.1, 723.4, 845.2, 1150.0, 1420.0\}\text{ Hz}$). Each oscillator frequency is modulated by `pitch` and progressively spread by `detune`:
   $$f_{mod, m} = f_m \times (1 + m \times \text{detune} \times 0.15) \times \text{pitch}$$

2. **Parallel Noise Source and Mix:**
   Combines the normalized metallic generator output with deterministic pseudo-random white noise using user-specified gain controls (`metal` and `noise`):
   $$\text{raw\_mix} = (\text{metal\_sample} \times \text{metal}) + (\text{noise\_sample} \times \text{noise})$$

3. **High-Pass Filtering (HPF):**
   Applies a 1-pole high-pass filter configured by `tone` cutoff frequency using RC time constant coefficient calculation:
   $$RC = \frac{1}{2\pi \times \text{tone}}, \quad \alpha = \frac{RC}{RC + dt}$$

4. **Envelope Generation & Click Suppression:**
   Calculates an exponential decay envelope ($\text{env\_factor} = \exp(-dt \times \frac{3}{\text{decay}})$). Applies a $2\text{ ms}$ linear attack ramp to eliminate initial DC offset clicks, followed by a final $3\text{ ms}$ buffer fade-out.

5. **Hyperbolic Tangent Saturation:**
   If `drive` exceeds $1.0$, the final filtered and enveloped signal is passed through hyperbolic tangent soft-clipping ($\text{std::tanh}(\text{sample} \times \text{drive})$) for added harmonic crunch and presence.