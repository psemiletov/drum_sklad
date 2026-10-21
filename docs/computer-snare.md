# Computer Snare Generator Documentation

The `computer_snare` generator is a snare drum synthesizer crafted in the style of early electronic music and minimal synth (Kraftwerk, 80s electro-pop). The sound is formed by pairing a square wave featuring a rapid pitch drop with filtered pseudo-random noise.

Call format:
```text
computer_snare(param1=val1, param2=val2, ...)
```

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`freq`** | `float` | `380.0` | `150.0` — `800.0` | Base frequency of the drum's tonal body (in Hz). |
| **`decay`** | `float` | `0.14` | `0.03` — `0.6` | Sound duration / decay time (in seconds). |
| **`crunch`** | `float` | `0.55` | `0.0` — `1.5` | Volume level of the noise layer (snare wires). |
| **`tone`** | `float` | `1100.0` | `200.0` — `8000.0` | Low-pass filter (LPF) cutoff frequency (in Hz) applied to the noise layer. |

---

## Architecture and Sonic Features

1. **Tonal Body:**
   Built using a square wave with a rapid exponential pitch envelope on the attack phase, creating a sharp, woody click.
2. **Noise Layer (Snare Wires):**
   Generated via fast bitwise XOR shifts (Xorshift algorithm) and processed through a single-pole LPF (`tone`) to model dry snare wire rattle.
3. **Saturation:**
   The final blend of tone and noise is saturated using `std::tanh` with a gain factor of `1.4`, giving the snare punch and cohesive density.
