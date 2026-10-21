# Kick Generator Documentation

The `render_kick` generator is a foundational percussion synthesis engine in Drumlabooh designed to model classic electronic and acoustic-style kick drum (bass drum) sounds. It utilizes a pure sine wave oscillator driven by an exponential pitch drop envelope, an independent exponential amplitude decay, a short white noise attack click for transient punch, and a smooth buffer fade-out stage.

Call format:
`kick(f0=val1, f1=val2, decay=val3, click=val4)`

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`f0`** | `float` | `150.0` | `50.0` — `400.0` | Initial starting frequency of the pitch drop envelope (in Hz). |
| **`f1`** | `float` | `45.0` | `20.0` — `100.0` | Final base target frequency of the pitch envelope (in Hz). |
| **`decay`** | `float` | `0.35` | `0.05` — `2.0` | Decay duration factor governing the overall tail length (in seconds). |
| **`click`** | `float` | `0.2` | `0.0` — `1.0` | Amplitude level of the short white noise attack transient. |

---

## Architecture and Sonic Features

1. **Exponential Pitch Drop & Sine Core:**
   - The instantaneous frequency is modulated by an exponential decay curve dropping from `f0` to `f1`:
     $$\text{freq} = f_1 + (f_0 - f_1) \cdot \exp(-t \cdot 20.0)$$
   - Phase accumulation generates a clean sine wave core:
     $$\text{sample} = \sin(\text{phase}) \cdot \text{env}$$

2. **Amplitude Envelope Decay:**
   - The amplitude is shaped by an exponential decay envelope linked to the `decay` parameter:
     $$\text{env} = \exp\left(-t \cdot \frac{3.0}{\text{decay}}\right)$$

3. **Attack Click & Fade-Out:**
   - A short white noise impulse click is injected during the first $5\text{ ms}$ ($t < 0.005\text{ s}$), linearly ramping down to simulate beater impact or initial transient punch.
   - A final $3\text{ ms}$ smoothing fade-out stage (`apply_buffer_fadeout`) is applied to the end of the buffer to eliminate any digital clicks or abrupt truncation.
