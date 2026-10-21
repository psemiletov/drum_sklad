# Boom Generator Documentation

The `boom` generator is a versatile percussion synthesizer featuring a solid low-end body, customizable saturation, dual-stage filtering, and optional granular micro-bursts. It is well-suited for crafting deep sub-basses, industrial hits, cybernetic Toms, and glitch percussion.

Call format:
```text
boom(param1=val1, param2=val2, ...)
```

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`duration`** | `float` | `0.35` | `0.05` — `2.0` | Total sound duration in seconds. |
| **`freq`** | `float` | `70.0` | `20.0` — `500.0` | Base starting frequency of the generator (in Hz). |
| **`sweep`** | `float` | `-40.0` | `-200.0` — `200.0` | Depth and direction of pitch envelope over time. |
| **`noise`** | `float` | `0.3` | `0.0` — `1.0` | Amount of noise component within the body of the sound. |
| **`roughness`**| `float` | `0.6` | `0.0` — `1.0` | Intensity of frequency wobble (detuning/roughness). |
| **`wobble`** | `float` | `24.0` | `0.0` — `100.0` | Speed (frequency in Hz) of the modulation wobble. |
| **`attack`** | `float` | `0.001` | `0.0` — `0.1` | Envelope attack time (in seconds). |
| **`release`** | `float` | `0.12` | `0.01` — `2.0` | Envelope release/decay time. |
| **`gain`** | `float` | `0.95` | `0.0` — `2.0` | Final output volume level. |
| **`bubble`** | `float` | `0.5` | `0.0` — `2.0` | Intensity of internal non-linear amplitude modulation (gives a resonant, springy character). |
| **`tone`** | `float` | `1400.0` | `100.0` — `10000.0`| Cutoff frequency of the output low-pass filter (LPF in Hz). |
| **`granular`** | `bool` | `true` | `0` / `1` | Enables the generation of random granular micro-bursts. |
| **`grain_mix`**| `float` | `0.4` | `0.0` — `1.0` | Mix balance of granular micro-bursts relative to the main signal. |

---

## Architecture and Sonic Features

1. **Pitch Sweep & Sub Layer:**
   Utilizes dual oscillators with distinct phases and dynamic pitch shifting (`kick_drop`), forming a massive low-frequency foundation.
2. **Filtering and Saturation:**
   The signal passes through a filter chain (an HPF set at 45 Hz to clear out subsonic rumble and a customizable LPF via `tone`), then gently saturates using `std::tanh` for density and compression.
3. **Granularity (`granular`):**
   When enabled, it randomly triggers short percussive clicks and micro-impulses, adding industrial texture and detail to the sound.
