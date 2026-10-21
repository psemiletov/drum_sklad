# Psytrance Kick (Psykick) Generator Documentation

The `render_psykick` generator is a specialized electronic percussion synthesis engine in Drumlabooh designed specifically for Psytrance, Goa, and fast hypnotic electronic music genres. It models the signature "psy-laser" punch by combining an exponential pitch drop envelope, a phase-modulation (FM) punch modifier, recursive amplitude decay, an aggressive waveshaper / hard-soft clipping distortion stage, a high-pass filtered Xorshift transient click, and a clean buffer fade-out stage.

Call format:
`psykick(f0=val1, f1=val2, decay=val3, click=val4, drive=val5, curve=val6, fm=val7, click_hp=val8)`

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`f0`** | `float` | `240.0` | `80.0` — `500.0` | Initial starting frequency of the pitch drop envelope (in Hz). |
| **`f1`** | `float` | `38.0` | `20.0` — `80.0` | Final baseline sub-frequency of the kick body (in Hz). |
| **`decay`** | `float` | `0.22` | `0.05` — `1.0` | Overall envelope duration factor governing the tail length (in seconds). |
| **`click`** | `float` | `0.25` | `0.0` — `1.0` | Amplitude level of the high-pass filtered noise transient click. |
| **`drive`** | `float` | `0.5` | `0.0` — `1.0` | Waveshaper and clipping distortion intensity for harmonic density and punch. |
| **`curve`** | `float` | `35.0` | `5.0` — `100.0` | Pitch envelope steepness determining how fast the frequency drops. |
| **`fm`** | `float` | `0.3` | `0.0` — `2.0` | Phase modulation depth introducing the signature "psy-laser" attack character. |
| **`click_hp`** | `float` | `3000.0` | `500.0` — `8000.0` | Cutoff frequency of the high-pass filter shaping the transient click (in Hz). |

---

## Architecture and Sonic Features

1. **Steep Pitch Drop & Phase Modulation (FM):**
   - The instantaneous frequency is modulated by an exponential decay curve governed by `curve`:
     $$\text{freq} = f_1 + (f_0 - f_1) \cdot \exp(-t \cdot \text{curve})$$
   - Phase accumulation integrates a dedicated FM modulator (`fm_mod`) during the initial hit to produce the signature laser-like psytrance snap:
     $$\text{phase} += 2.0 \cdot \pi \cdot (\text{freq} + \text{fm\_mod}) \cdot \text{dt}$$

2. **Recursive Amplitude Decay & Waveshaper Distortion:**
   - Amplitude is shaped efficiently via a recursive decay factor (`env *= env_factor`).
   - The driven signal passes through an aggressive waveshaper combining pre-gain ($1.0 + \text{drive} \cdot 6.0$), hard clipping, and linear interpolation via `juce::jmap` for maximum energy and loudness.

3. **High-Pass Filtered Transient Click & Fade-Out:**
   - Xorshift noise generated during the first $4\text{ ms}$ ($t < 0.004\text{ s}$) passes through a 1-pole high-pass filter (`hp_alpha`) configured via `click_hp` to ensure crisp transient definition without low-end mud.
   - A final $3\text{ ms}$ smoothing fade-out (`apply_buffer_fadeout`) finishes the buffer cleanly.
