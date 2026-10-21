# Glass / Crystalline Percussion Generator Documentation

The `render_glass` generator is a specialized crystalline percussion and glass-like synthesis engine in Drumlabooh designed for brittle, resonant, and glassy hits, tuned metallic effects, or percussive bells. It utilizes optimized `fast_sin` oscillators driven by golden-ratio inharmonic intervals, an Xorshift noise-to-glass blend, a resonant band-pass filter stage ($Q = 1.8$), a recursive body envelope factor, a sharp glass-strike attack transient (`ping`), a linear release tail fade-out, and a soft-clipping saturation circuit.

Call format:
`glass(decay=val1, pitch=val2, tone=val3, drive=val4, noise=val5, ping=val6)`

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`decay`** | `float` | `1.8` | `0.1` — `8.0` | Overall envelope duration factor governing tail length (in seconds). |
| **`pitch`** | `float` | `1.1` | `0.3` — `3.0` | Global pitch tuning multiplier applied to the base glass frequency. |
| **`tone`** | `float` | `0.6` | `0.0` — `1.0` | Brightness and filter cutoff frequency. Values $\le 1.0$ map automatically to $[3000.0, 12000.0]\text{ Hz}$. |
| **`drive`** | `float` | `1.1` | `1.0` — `5.0` | Soft-clipping saturation multiplier applied via `std::tanh`. |
| **`noise`** | `float` | `0.25` | `0.0` — `1.0` | Balance ratio between raw Xorshift noise and the inharmonic `fast_sin` ring cluster. |
| **`ping`** | `float` | `0.6` | `0.0` — `2.0` | Intensity level of the sharp, crystalline glass-strike attack transient. |

---

## Architecture and Sonic Features

1. **Inharmonic Glass Ring & Fast Sine Oscillators:**
   - Six oscillators use `fast_sin` lookup/approximations driven by golden-ratio inharmonic intervals ($1.000\times$, $1.618\times$, $2.542\times$, $3.833\times$, $5.211\times$, $7.145\times$) based on `base_glass` ($1200.0\text{ Hz} \cdot \text{pitch}$):
     $$\text{glass\_ring} = (\text{osc}_1 \cdot \text{osc}_3 + \text{osc}_2 \cdot \text{osc}_4 + \text{osc}_5 \cdot \text{osc}_6) \cdot 0.25$$

2. **Resonant Band-Pass Filtering:**
   - The combined noise and glass signal passes through a band-pass filter with higher resonance ($Q = 1.8$) compared to standard metal cymbal generators, emphasizing brittle, resonant glass formants.

3. **Recursive Body Envelope & Glass-Strike Attack (`Ping`):**
   - Amplitude decay is calculated recursively per sample (`body_env *= body_env_factor`) for efficient exponential decay.
   - A short $1.2\text{ ms}$ attack transient ($t < 0.0012\text{ s}$) injects a sharp, brittle crack scaled by `ping`.

4. **Release Tail Fade-Out & Saturation:**
   - A linear release fade (`release_fade`) is applied across the last $25\%$ of the decay duration (`tail_time = decay * 0.25`).
   - The final signal passes through `std::tanh` when `drive > 1.0` to control peaks and add harmonic density.
