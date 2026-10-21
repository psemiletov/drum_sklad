```markdown

# Rimshot Generator Documentation

The `rimshot` generator is a specialized percussion synthesis engine in Drumlabooh designed for creating crisp drum rimshots and cross-stick hits. It combines a high-frequency resonant tonal body with a filtered pseudo-random noise click transient, envelope shaping, and optional saturation drive.

Call format:

```text
rimshot(freq=val1, decay=val2, click=val3, tone=val4, drive=val5)

```

## Parameters Table

| Parameter | Type | Default | Range | Description |
| --- | --- | --- | --- | --- |
| **`freq`** | `float` | `440.0` | `150.0` — `1200.0` | Base fundamental resonance frequency of the rim/shell body (in Hz). |
| **`decay`** | `float` | `0.12` | `0.02` — `1.0` | Overall duration and tail decay factor (in seconds). |
| **`click`** | `float` | `0.7` | `0.0` — `2.0` | Intensity level of the filtered stick-impact noise transient. |
| **`tone`** | `float` | `2500.0` | `800.0` — `8000.0` | Cutoff frequency of the low-pass filter shaping the click transient (in Hz). |
| **`drive`** | `float` | `1.4` | `0.5` — `5.0` | Saturation overdrive multiplier processed via `std::tanh`. |

---

## Architecture and Sonic Features

1. **Resonant Body with Pitch Drop:**
The tonal component models the sharp shell/rim resonance using a sine wave oscillator with a fast exponential pitch drop from an elevated initial transient down toward `freq`:

$$\text{current\_freq} = \text{freq} + \text{freq} \cdot 0.8 \cdot \exp(-t \cdot 80.0)$$



Its amplitude envelope decays exponentially based on `decay`.
2. **Filtered Noise Transient Click:**
A deterministic Xorshift pseudo-random generator (`rim_noise_state = 54321`) produces raw noise filtered through a single-pole RC low-pass filter governed by `tone`. During the first $3\text{ ms}$, a linear drop creates a sharp stick impact snap, followed by a short exponential release tail.
3. **Saturation & Output Finishing:**
The combined body and click signals pass through an optional `std::tanh` saturation stage when $\text{drive} > 1.0$ to add harmonic crunch. A $3\text{ ms}$ fade-out smoothing stage (`apply_buffer_fadeout`) finishes the buffer.

```

---
