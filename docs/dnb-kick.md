# DnB Kick Generator Documentation

The `render_dnb_kick` generator is an aggressive percussion synthesis engine in Drumlabooh designed specifically for Drum and Bass, industrial, and hard electronic music genres. It features a fast exponential pitch drop for high-impact chest punch, a dedicated Xorshift PRNG transient click generator to cut through heavy mixes, an exponential envelope decay, and an aggressive industrial saturation stage driven by `std::tanh`.

Call format:
`dnb_kick(start_freq=val1, end_freq=val2, pitch_time=val3, decay=val4, click=val5, drive=val6)`

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`start_freq`** | `float` | `150.0` | `80.0` — `300.0` | Initial starting frequency of the pitch envelope (in Hz). |
| **`end_freq`** | `float` | `45.0` | `30.0` — `80.0` | Final baseline sub-frequency of the kick body (in Hz). |
| **`pitch_time`** | `float` | `0.035` | `0.01` — `0.1` | Time constant governing the speed of the pitch drop (impact sharpness, in seconds). |
| **`decay`** | `float` | `0.35` | `0.05` — `2.0` | Overall envelope duration factor governing tail length (in seconds). |
| **`click`** | `float` | `1.5` | `0.0` — `5.0` | Intensity level of the noise-based attack transient click. |
| **`drive`** | `float` | `2.0` | `1.0` — `10.0` | Industrial overdrive saturation multiplier passed through `std::tanh`. |

---

## Architecture and Sonic Features

1. **Fast Exponential Pitch Envelope & Sub Core:**
   - Instantaneous frequency drops rapidly from `start_freq` to `end_freq` based on `pitch_time`:
     $$\text{freq} = \text{end\_freq} + (\text{start\_freq} - \text{end\_freq}) \cdot \exp\left(-\frac{t}{\text{pitch\_time}}\right)$$
   - The sub-bass core is generated via sine wave phase accumulation and shaped by an exponential amplitude decay:
     $$\text{sub} = \sin(\text{phase}) \cdot \exp\left(-t \cdot \frac{3.0}{\text{decay}}\right)$$

2. **Xorshift Transient Click:**
   - A high-intensity noise transient is generated via an internal Xorshift PRNG state machine.
   - Active only during the first $4\text{ ms}$ ($t < 0.004\text{ s}$), linearly ramping down to give the kick the sharp click needed to cut through heavily compressed DnB mixes.

3. **Industrial Saturation Drive & Fade-Out:**
   - The mixed sub and transient signal passes through a non-linear `std::tanh` saturation stage multiplied by `drive` to introduce harmonic grit and heavy clipping.
   - A final $5\text{ ms}$ smoothing fade-out stage (`apply_buffer_fadeout`) cleans up the tail.
