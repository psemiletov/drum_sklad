# Snare Generator Documentation

The `snare` generator is a flexible synthesis engine in Drumlabooh designed for creating acoustic and electronic snare drum sounds. It combines a frequency-modulated tonal body component with a filtered, shaped noise component simulating the snare wires.

Call format:
```text
snare(f0=val1, f1=val2, decay=val3, noise=val4, n_decay=val5)
```

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`f0`** | `float` | `200.0` | `80.0` — `500.0` | Initial starting frequency of the tonal body component (in Hz). |
| **`f1`** | `float` | `110.0` | `50.0` — `300.0` | Target fundamental bottom frequency of the tonal body component (in Hz). |
| **`decay`** | `float` | `0.18` | `0.05` — `1.0` | Duration of the tonal body envelope decay (in seconds). |
| **`noise`** | `float` | `0.6` | `0.0` — `2.0` | Relative mix level of the filtered snare wires noise component. |
| **`n_decay`** | `float` | `0.12` | `0.02` — `1.0` | Duration of the noise envelope decay (in seconds). |

---

## Architecture and Sonic Features

1. **Tonal Body Synthesis & Pitch Drop:**
   The body component models the drum shell resonance using a sine oscillator with a fast exponential pitch drop. Frequency transitions from `f0` down toward `f1` following the curve:
   $$\text{freq} = f_1 + (f_0 - f_1) \cdot \exp(-t \cdot 45.0)$$
   The body amplitude decays exponentially based on the `decay` parameter.

2. **Snare Wires Noise & Filtering:**
   The wire component utilizes a deterministic Xorshift pseudo-random generator (`snare_noise_state = 12345`). The raw noise passes through a single-pole RC low-pass filter fixed at a cutoff frequency of $2500\text{ Hz}$ to simulate crisp wire rattle. A linear ramp-up over the first $1.5\text{ ms}$ prevents initial clicks in the noise envelope.

3. **Output Saturation & Finishing:**
   The mixed body and noise signals are passed through a soft-clipping `std::tanh` saturation stage with an input drive multiplier of $1.3$. Finally, a short crossfade fade-out is applied to the end of the buffer via `apply_buffer_fadeout`.
