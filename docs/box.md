# Box Sound (Fart / Pressure Release) Generator Documentation

The `box_sound` generator is an organic, physiological synthesis engine in Drumlabooh designed for rendering air pressure release, tissue vibration, and fart sound effects. It combines a frequency-modulated sub-bass core with vibrato, integrated brown noise, transient high-frequency squeaks, and pressure-dependent dynamic low-pass filtering.

Call format:
```text
box_sound(duration=val1, freq=val2, sweep=val3, noise=val4, bass=val5, vibrato=val6, squeak=val7, attack=val8, release=val9, gain=val10)
```

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`duration`** | `float` | `0.32` | `0.05` — `2.0` | Total length of the sound event (in seconds). |
| **`freq`** | `float` | `55.0` | `20.0` — `200.0` | Base fundamental frequency of tissue vibration (in Hz). |
| **`sweep`** | `float` | `-25.0` | `-100.0` — `100.0` | Pitch shift amount across the duration (in Hz). |
| **`noise`** | `float` | `0.7` | `0.0` — `2.0` | Mix level of integrated brown noise (airflow rumble). |
| **`bass`** | `float` | `0.4` | `0.0` — `2.0` | Sub-bass fundamental and harmonic resonance level. |
| **`vibrato`** | `float` | `22.0` | `0.0` — `60.0` | Rate of high-speed frequency modulation / sphincter flutter (in Hz). |
| **`squeak`** | `float` | `0.25` | `0.0` — `1.0` | Mix level of the high-pitched opening air jet squeak. |
| **`attack`** | `float` | `0.002` | `0.001` — `0.05` | Valve opening attack time (in seconds). |
| **`release`** | `float` | `0.14` | `0.01` — `1.0` | Pressure decay time constant (in seconds). |
| **`gain`** | `float` | `0.95` | `0.0` — `2.0` | Final output level scaling multiplier. |

---

## Architecture and Sonic Features

1. **Sub-Bass & Vibrato Flutter:**
   The body layer models membrane and tissue vibration using a sine wave plus its second harmonic ($\sin(\text{phase}) + 0.4 \cdot \sin(2 \cdot \text{phase})$). Rapid vibrato at `vibrato_r` ($22\text{ Hz}$) with $50\%$ depth simulates high-speed flutter under pressure.

2. **Brown Noise Airflow & Transient Squeak:**
   - Deterministic white noise is integrated into leaky brown noise ($\text{brown} = \text{brown} \cdot 0.92 + \text{white} \cdot 0.28$) to produce low-end air turbulence.
   - For the first $40\%$ of the sound duration ($u < 0.4$), a noise-modulated high frequency sine wave ($1800\text{ Hz} - 2400\text{ Hz}$) adds a realistic opening squeak/hiss.

3. **Dynamic Pressure Filtering & Saturation:**
   - A 1-pole dynamic LPF sweeps between $120\text{ Hz}$ and $900\text{ Hz}$ based on air pressure (`pressure_env`), opening up during peak pressure.
   - A static $35\text{ Hz}$ HPF removes DC offsets and sub-rumble clutter.
   - The combined signal is driven through a soft `std::tanh` analog saturator (multiplier $2.5$) and final $3\text{ ms}$ fade-out smoothing.
