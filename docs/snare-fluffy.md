# Snare Fluffy Generator Documentation

The `render_snare_fluffy` generator is an advanced and harmonically rich snare drum synthesis engine in Drumlabooh. It combines a dual-harmonic tonal body shell with attack FM modulation, a diffusive snare wire noise path equipped with a micro-delay resonator, exponential decay envelopes, and a saturation stage to deliver a dense, punchy, and lively acoustic snare character.

Call format:
`snare_fluffy(body_freq=val1, decay=val2, snap=val3, fm=val4, tone=val5, drive=val6)`

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`body_freq`** (or `freq`) | `float` | `190.0` | `80.0` — `400.0` | Primary base frequency of the tonal drum body shell (in Hz). |
| **`decay`** | `float` | `0.22` | `0.05` — `1.5` | Overall envelope decay duration factor governing the tail length (in seconds). |
| **`snap`** | `float` | `2.2` | `0.1` — `5.0` | Intensity, density, and tail level of the noisy snare wires. |
| **`fm`** | `float` | `1.5` | `0.0` — `5.0` | Aggressiveness and depth of the attack frequency modulation click. |
| **`tone`** | `float` | `4200.0` | `1000.0` — `10000.0` | Cutoff frequency of the 1-pole low-pass filter shaping the noise timbre (in Hz). |
| **`drive`** | `float` | `2.0` | `1.0` — `5.0` | Density and non-linear saturation coefficient (`std::tanh`). |

---

## Architecture and Sonic Features

1. **Dual-Harmonic Body & FM Attack:**
   - The oscillator frequency is modulated by a fast pitch envelope dropping from its initial peak:
     $$\text{cur\_freq} = \text{body\_freq} + (\text{body\_freq} \cdot 0.7 \cdot \exp(-t \cdot 60.0))$$
   - FM modulation acts during the initial milliseconds, introducing a metallic or wooden attack transient via phase shifting (`fm_mod`) driven by `fm_env`.
   - The body is synthesized from two phase harmonics (fundamental and a $1.6\times$ overtone) multiplied by an exponential amplitude decay envelope:
     $$\text{body} = (\sin(\text{phase}_1) \cdot 0.6 + \sin(\text{phase}_2) \cdot 0.3) \cdot \exp\left(-t \cdot \frac{5.5}{\text{decay}}\right)$$

2. **Diffusive Snare Wires & Micro-Resonator:**
   - Pseudo-random Xorshift noise passes through a 1-pole RC low-pass filter (`tone`) and feeds a $2\text{ ms}$ ring delay buffer, introducing spatial diffusion and a characteristic "fluffy" snare texture.
   - The wire envelope (`snap_env`) boosts the initial transient by $2.2\times$ during the first $3\text{ ms}$ ($t < 0.003\text{ s}$) to ensure a sharp stick crack.

3. **Saturation & Tail Fade-Out:**
   - The mixed shell and wire signal undergoes soft-to-hard clipping via `std::tanh` whenever `drive > 1.0`.
   - A final smoothing stage (`apply_buffer_fadeout`, $4\text{ ms}$) cleanly terminates the audio buffer.
