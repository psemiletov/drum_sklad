# Kick Generator Documentation

The `kick` generator is a standard, versatile electronic bass drum synthesis engine in Drumlabooh. It creates clean and punchy acoustic-style or electronic kicks using a frequency-modulated sine oscillator coupled with a short noise-based click transient.

Call format:
```text
kick(f0=val1, f1=val2, decay=val3, click=val4)
```

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`f0`** | `float` | `150.0` | `60.0` — `400.0` | Initial pitch frequency at the attack transient (in Hz). |
| **`f1`** | `float` | `45.0` | `30.0` — `100.0` | Target fundamental frequency of the body tail (in Hz). |
| **`decay`** | `float` | `0.35` | `0.1` — `1.0` | Overall envelope decay duration governing the low-end tail length (in seconds). |
| **`click`** | `float` | `0.2` | `0.0` — `1.0` | Relative amplitude level of the initial noise click transient. |

---

## Architecture and Sonic Features

1. **Pitch Sweep & Sinusoidal Body:**
   The fundamental body frequency drops exponentially from `f0` down to `f1` over time using the rate curve:
   $$\text{freq} = f_1 + (f_0 - f_1) \cdot \exp(-t \cdot 20.0)$$
   The amplitude envelope decays smoothly based on the `decay` parameter:
   $$\text{env} = \exp\left(-t \cdot \frac{3.0}{\text{decay}}\right)$$

2. **Transient Click Generation:**
   During the first $5\text{ ms}$ ($t < 0.005\text{ s}$), a pseudo-random white noise click is added to the signal, linearly fading out to simulate beater impact or stick snap.

3. **Buffer Finalization:**
   The generated samples are written directly to the buffer, followed by a $3\text{ ms}$ smoothing fade-out applied via `apply_buffer_fadeout` to prevent clicking artifacts at the end of the buffer.
