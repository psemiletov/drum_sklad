# Gabber Kick Generator Documentation

The `gabber_kick` generator is a specialized synthesizer engineered for producing heavy, distorted kick drums characteristic of Gabber, Hardcore, Terror, and Rave electronic music genres. It combines an exponential pitch envelope with dynamic wavefolding overdrive and low-pass filtering.

Call format:
```text
gabber_kick(param1=val1, param2=val2, ...)
```

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`f0`** | `float` | `220.0` | `100.0` — `500.0` | Initial transient start frequency (in Hz) providing the impact click. |
| **`f1`** | `float` | `42.0` | `20.0` — `100.0` | Base fundamental sub-frequency (in Hz) reached at the end of the pitch drop. |
| **`decay`** | `float` | `0.38` | `0.05` — `1.5` | Decay time (in seconds) determining the overall tail length of the kick. |
| **`drive`** | `float` | `0.8` | `0.0` — `2.0` | Distortion intensity factor controlling the saturation envelope. |
| **`tone`** | `float` | `500.0` | `100.0` — `5000.0` | Cutoff frequency of the internal single-pole low-pass filter (in Hz). |

---

## Architecture and Sonic Features

1. **Exponential Pitch Sweep:**
   The base sine wave rapidly drops from the initial attack frequency (`f0`) down to the sub-bass fundamental (`f1`) according to an exponential curve (`std::exp(-t * 32.0f)`), creating a sharp percussive punch.
2. **Time-Decaying Distortion:**
   Drive gain peaks at the impact phase and exponentially decays over time (`std::exp(-t * 15.0f)`). The boosted signal is shaped using hyperbolic tangent saturation (`std::tanh`), yielding maximum overdrive grit at the transient while preserving a cleaner low-end tail.
3. **Tone Shaping & Parallel Mix:**
   The distorted signal is routed through a low-pass filter defined by `tone` and blended back with 30% dry sine wave (`raw_sine * 0.3f + lpf_state * 0.7f`), retaining clarity and presence in high frequencies before final soft-clipping.
