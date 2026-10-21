# Psytrance Kick Generator Documentation

The `psytrance_kick` generator is a versatile kick drum synthesizer designed to produce precise, tight, and punchy bass kicks suitable for Psytrance, Goa, Techno, EDM, and modern electronic dance music. It combines a clean sinusoidal pitch-drop sub-body with a brief Xorshift noise-burst transient click for maximum mix punch and clarity.

Call format:
```text
psytrance_kick(param1=val1, param2=val2, ...)
```

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`f0`** | `float` | `240.0` | `100.0` — `600.0` | Initial transient start frequency (in Hz) determining the attack pitch. |
| **`f1`** | `float` | `38.0` | `20.0` — `100.0` | Sub-bass fundamental frequency (in Hz) reached after the pitch drop. |
| **`decay`** | `float` | `0.22` | `0.05` — `1.5` | Decay time and body tail length of the kick drum (in seconds). |
| **`click`** | `float` | `0.35` | `0.0` — `1.0` | Mix level and intensity of the initial noise-burst transient click. |

---

## Architecture and Sonic Features

1. **Exponential Pitch Sweep:**
   The fundamental sine wave drops exponentially from `f0` down to `f1` (`std::exp(-t * 35.0f)`), creating a focused, tight low-end punch without muddy sub-rumble.
2. **Sinusoidal Sub-Body:**
   The body relies on a pure sine wave multiplied by a smooth exponential amplitude decay envelope (`env_factor`), delivering a clean, phase-consistent low-end response.
3. **Transient Noise Click:**
   During the first 4 milliseconds (`t < 0.004f`), a linear-decaying burst of pseudo-random Xorshift noise (`click_noise_state = 9999`) is blended into the attack phase. This ensures a crisp top-end click that easily cuts through dense basslines and heavy synth mixes.
