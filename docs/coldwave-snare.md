# Coldwave Snare Generator Documentation

The `coldwave_snare` generator is a specialized synthesizer tailored for creating authentic snare drums characteristic of Coldwave, Post-Punk, and 80s Gothic Rock. It pairs an adjustable tonal pitch drop with an Xorshift pseudo-random noise layer and low-pass filtering, delivering a dry, gloomy, and boxy character typical of classic drum machines and early digital samplers.

Call format:
```text
coldwave_snare(param1=val1, param2=val2, ...)
```

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`f0`** | `float` | `210.0` | `80.0` — `400.0` | Initial pitch start frequency (in Hz) for the snare body transient. |
| **`f1`** | `float` | `115.0` | `50.0` — `250.0` | Base fundamental end frequency (in Hz) after the pitch drop. |
| **`decay`** | `float` | `0.17` | `0.05` — `0.6` | Overall ring time and duration of the snare (in seconds). |
| **`crunch`** | `float` | `0.5` | `0.0` — `1.5` | Volume mix level and texture intensity of the noise layer (snare wires). |
| **`tone`** | `float` | `900.0` | `200.0` — `5000.0` | Low-pass filter (LPF) cutoff frequency (in Hz) applied to the noise layer. |

---

## Architecture and Sonic Features

1. **Tonal Pitch Sweep:**
   The internal oscillator rapidly glides from `f0` down to `f1` following an exponential curve (`std::exp(-t * 40.0f)`), simulating the characteristic acoustic resonance and tight membrane tension of minimalist drum machines.
2. **Deterministic Noise Layer (Xorshift):**
   Noise is generated via a fixed-seed Xorshift algorithm (`cw_noise_state = 43210`), ensuring consistent, repeatable character without randomized phase washing.
3. **Filtering & Saturation:**
   The noise passes through a single-pole low-pass filter defined by `tone` with a brief ramp-up envelope on the attack (`t < 0.002f`). The combined signal is then glued together using hyperbolic tangent soft-clipping (`std::tanh`), achieving a gritty, lo-fi post-punk aesthetic.
