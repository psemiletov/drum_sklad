# Fart Synthesizer (`fart`) Documentation / Документация синтезатора `fart`

---

## English Documentation (`FART_SYNTH.md`)

The `fart` synthesis engine models the physical acoustics of pressurized air releases through a flexible orifice. It combines a low-pass filtered noise burst sequence with a low-frequency pitch-dropping sub-oscillator and non-linear saturation to generate a wide variety of sounds—from dry, tight pops to deep, wet, sub-heavy bursts.

### Formula Syntax

Preset Name Index^fart(decay=FLOAT, tone=FLOAT, spread=FLOAT, count=INT, pitch=FLOAT, wet=FLOAT)


### Parameters

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| `decay` | Float | `0.45` | `0.1 – 1.5` | **Total duration** of the sound in seconds. Shorter values give tight pops; longer values create sustained rumbles. |
| `tone` | Float | `650.0` | `80.0 – 3000.0` | **Low-pass filter cutoff frequency** (Hz). Lower values result in deep, muffled sounds, while higher values add bright, squeaky high-frequency content. |
| `spread` | Float | `0.022` | `0.005 – 0.060` | **Time gap between burst impulses** in seconds. Controls the rate of the "flutter" or "flapping" texture. |
| `count` | Int | `8` | `1 – 30` | **Number of discrete pressure impulses** per trigger. Higher counts create longer, multi-stage, ripple-like bursts. |
| `pitch` / `freq` | Float | `45.0` | `10.0 – 150.0` | **Sub-bass resonance frequency** (Hz). Controls the fundamental tone of the body movement, featuring an exponential downward pitch drop at the attack. |
| `wet` | Float | `1.2` | `0.0 – 3.5` | **Saturation / Drive intensity**. Applies non-linear hyperbolic tangent (`tanh`) distortion to increase density, wetness, and punch. |

### Parameter Tuning Examples

* **Dry & Tight Pop:** `decay=0.18, tone=850.0, spread=0.010, count=3, pitch=60.0, wet=0.7`
* **Deep Sub Rumble:** `decay=0.90, tone=150.0, spread=0.045, count=18, pitch=16.0, wet=2.8`
* **High-Pitched Squeak:** `decay=0.25, tone=1800.0, spread=0.008, count=4, pitch=110.0, wet=1.0`
* **Juicy / Wet Burst:** `decay=0.55, tone=320.0, spread=0.028, count=12, pitch=28.0, wet=2.4`
