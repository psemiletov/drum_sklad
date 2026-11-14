# Hybrid Crash Cymbal Generator Documentation

The `render_crash_hybrid` generator is an advanced hybrid crash cymbal synthesis engine in Drumlabooh. It builds upon metallic synthesis principles by pairing a lower-tuned 6-oscillator inharmonic metal core with a bright, silky high-frequency noise wash engine. It features normalized/Hz parameter mapping, dual-stage noise filtering (LPF + HF emphasis), a main High-Pass Filter (HPF), a attack-transient envelope with custom tail scaling, non-linear saturation, and explicit IEEE floating-point safety checks (`std::isfinite`).

Call format:
`crash_hybrid(decay=val1, tone=val2, pitch=val3, drive=val4, noise=val5)`

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`decay`** | `float` | `1.5` | `0.1` — `6.0` | Overall decay duration of the cymbal envelope tail (in seconds). |
| **`tone`** | `float` | `5500.0` | `500.0` — `samplerate * 0.45` | Cutoff frequency of the primary HPF (in Hz). Accepts normalized range $[0.0, 1.0]$, mapped automatically to $[2000.0, 10000.0]\text{ Hz}$. |
| **`pitch`** | `float` | `1.0` | `0.3` — `3.0` | Global pitch tuning multiplier applied to all metallic oscillators. |
| **`drive`** | `float` | `1.2` | `1.0` — `5.0` | Non-linear saturation multiplier passed through `std::tanh` for harmonic density. |
| **`noise`** | `float` | `0.5` | `0.0` — `1.0` | Balance control between metallic square wave core ($1 - \text{noise}$) and processed noise wash ($\text{noise}$). |

---

## Architecture and Sonic Features

1. **Inharmonic Metallic Core:**
   - Uses six distinct base frequencies tuned lower than standard hi-hats ($205.3\text{ Hz}$, $304.4\text{ Hz}$, $369.6\text{ Hz}$, $522.7\text{ Hz}$, $540.0\text{ Hz}$, $800.0\text{ Hz}$) to simulate heavy bronze body resonance:
     $$\text{metal\_sample} = \frac{1}{6} \sum_{m=0}^{5} \text{sq}(\text{phase}_m \cdot \text{pitch})$$

2. **Silky High-Frequency Noise Wash Engine:**
   - Xorshift noise is split via a fast single-pole LPF ($\approx 2.6\text{ kHz}$ cutoff, $\text{dt} / (0.00006 + \text{dt})$).
   - High frequencies are isolated via difference subtraction ($\text{noise\_diff} = \text{raw\_noise} - \text{noise\_lpf\_state}$) and re-blended to boost top-end air and shimmer:
     $$\text{noise\_sample} = (\text{noise\_lpf\_state} \cdot 0.7) + (\text{noise\_diff} \cdot 0.8)$$

3. **Hybrid Crossfade & Primary HPF:**
   - Smooth linear blend between metal and noise:
     $$\text{raw\_mix} = \text{metal\_sample} \cdot (1.0 - \text{noise\_mix}) + \text{noise\_sample} \cdot \text{noise\_mix}$$
   - Filtered through a 1-pole high-pass filter to strip away low rumble and clarify the high spectrum.

4. **Transient Envelope & Safety Features:**
   - Initial $3\text{ ms}$ stick-impact attack boost ($2.5\times$) followed by an exponential decay tail scaled by tail progress:
     $$\text{env} = \exp\left(-2.5 \cdot \frac{t}{\text{decay}}\right) \cdot (1.0 - 0.15 \cdot \text{tail\_progress})$$
   - Includes NaN/Inf safety protection (`!std::isfinite(sample)`) and a $20\text{ ms}$ tail fade-out (`apply_buffer_fadeout`).
