# Cowbell Generator Documentation

The `cowbell_sound` generator is a metallic percussion synthesis engine in Drumlabooh modeled after classic analog drum machine cowbells (such as the TR-808 cowbell). It utilizes two detuned square wave oscillators, single-pole low-pass filtering, an attack click, exponential envelope decay, and optional saturation drive.

Call format:
```text
cowbell_sound(freq=val1, ratio=val2, decay=val3, tone=val4, drive=val5)
```

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`freq`** | `float` | `560.0` | `200.0` — `1500.0` | Fundamental base frequency of the primary oscillator (in Hz). |
| **`ratio`** | `float` | `1.45` | `1.0` — `2.5` | Frequency multiplier for the second detuned oscillator. |
| **`decay`** | `float` | `0.18` | `0.02` — `1.0` | Exponential decay duration (in seconds) governing tail length. |
| **`tone`** | `float` | `1200.0` | `400.0` — `8000.0` | Cutoff frequency of the 1-pole RC low-pass filter (in Hz). |
| **`drive`** | `float` | `1.2` | `0.5` — `5.0` | Overdrive saturation multiplier passed through `std::tanh`. |

---

## Architecture and Sonic Features

1. **Dual Square Wave Oscillators:**
   Two square wave oscillators generate the classic metallic inharmonic timbre:
   - Oscillator 1 runs at $f_1 = \text{freq}$ (default $560\text{ Hz}$).
   - Oscillator 2 runs at $f_2 = \text{freq} \cdot \text{ratio}$ (default $812\text{ Hz}$ at ratio $1.45$).
   
   The raw square waves ($\pm 1.0$) are mixed equally ($50\% / 50\%$).

2. **Tone Filtering & Attack Click:**
   - The mixed square wave signal passes through a single-pole RC low-pass filter defined by `tone`:
     $$\alpha = \frac{dt}{rc + dt}, \quad rc = \frac{1}{2\pi \cdot \text{tone}}$$
   - A short impulse click is added during the first $1.5\text{ ms}$ ($t < 0.0015\text{ s}$) to emphasize stick impact.

3. **Envelope Decay & Saturation Drive:**
   The filtered signal with click is modulated by an exponential decay envelope:
   $$\text{env} = \exp\left(-t \cdot \frac{4.0}{\text{decay}}\right)$$
   When $\text{drive} > 1.0$, the sound is passed through a soft-clipping `std::tanh(\text{sample} \cdot \text{drive})` stage for added warmth and harmonic distortion, followed by a $3\text{ ms}$ fade-out smoothing stage (`apply_buffer_fadeout`).
