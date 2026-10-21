# Clap Generator Documentation

The `clap` generator is a flexible synthesis engine in Drumlabooh designed for rendering handclap percussion sounds. It mimics human handclaps using a multi-burst transient phase followed by a noise tail, conditioned through a single-pole low-pass filter.

Call format:
```text
clap(decay=val1, tone=val2, spread=val3, count=val4)
```

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`decay`** | `float` | `0.28` | `0.05` — `1.0` | Main envelope decay duration (in seconds) governing the tail length. |
| **`tone`** | `float` | `1200.0` | `400.0` — `5000.0` | Cutoff frequency of the single-pole low-pass filter (in Hz) shaping brightness. |
| **`spread`** | `float` | `0.011` | `0.003` — `0.03` | Time offset delay (in seconds) between successive micro-bursts during the attack. |
| **`count`** | `int` | `4` | `1` — `8` | Total number of initial micro-bursts simulating individual hand impacts. |

---

## Architecture and Sonic Features

1. **Multi-Burst Transient Generation:**
   The attack phase simulates several hands clapping at slightly staggered moments. A loop generates `count` overlapping exponential decay envelopes ($\exp(-dt_{burst} \cdot 400)$), spaced apart in time by the `spread` factor (default $11\text{ ms}$).

2. **Tail Decay & Envelope Blending:**
   An overall exponential tail envelope decays across the sound duration ($\exp(-dt \cdot \frac{3.5}{decay})$). The total envelope combines both phases:
   $$\text{total\_env} = (\text{burst\_env} \cdot 0.8) + (\text{tail\_env} \cdot 0.2)$$
   This provides a sharp multi-impact attack along with a smooth, sustained reverberant noise tail.

3. **Deterministic Noise & Low-Pass Filtering:**
   The noise engine uses a deterministic pseudo-random sequence (`noise_state = 77777`). The signal passes through a 1-pole RC low-pass filter defined by `tone`:
   $$\alpha = \frac{dt}{rc + dt}, \quad rc = \frac{1}{2\pi \cdot \text{tone}}$$
   Final samples are scaled by an overall gain multiplier of $1.2$ before an ending fade-out smoothing stage.
