# Goa Trance Kick (Goakick) Generator Documentation

The `render_goakick` generator is an authentic 90s Goa trance percussion synthesis engine in Drumlabooh designed to replicate classic hardware and early sampler-based kick drums (such as Akai S1000 workflows). It features a two-stage exponential pitch drop envelope, sine-to-triangle wave morphing via `warmth`, a short tonal acoustic beater thump ("kolotushka"), asymmetrical Mackie mixing console saturation, and a selectable bit-depth DAC quantization stage for vintage lo-fi character.

Call format:
`goakick(f0=val1, f1=val2, decay=val3, punch=val4, warmth=val5, bit_depth=val6)`

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`f0`** | `float` | `220.0` | `80.0` — `400.0` | Initial starting frequency of the pitch envelope (in Hz). |
| **`f1`** | `float` | `48.0` | `30.0` — `80.0` | Final base target frequency of the kick body (in Hz). |
| **`decay`** | `float` | `0.28` | `0.05` — `1.0` | Overall envelope duration factor governing tail length (in seconds). |
| **`punch`** | `float` | `0.40` | `0.0` — `1.0` | Amplitude level of the short tonal acoustic beater thump. |
| **`warmth`** | `float` | `0.35` | `0.0` — `1.0` | Controls sine-to-triangle wave harmonic morphing and asymmetrical console saturation. |
| **`bit_depth`** | `float` | `16.0` | `8.0` — `16.0` | DAC bit-depth quantization emulation for vintage Akai S1000 lo-fi grit. |

---

## Architecture and Sonic Features

1. **Exponential Pitch Drop & Waveform Morphing:**
   - Frequency drops exponentially from `f0` to `f1` based on a fast pitch envelope factor:
     $$\text{freq\_body} = f_1 + (f_0 - f_1) \cdot \exp(-t \cdot 28.0)$$
   - The body mixes pure sine wave and triangle wave forms (derived via $\frac{2}{\pi} \arcsin(\sin(\text{phase}))$) scaled by `warmth`:
     $$\text{body\_signal} = \text{jmap}(\text{warmth}, 0.0, 1.0, \sin(\text{phase}), \text{tri\_wave}) \cdot \text{env}$$

2. **Acoustic Beater Thump (`Punch`):**
   - Active during the first $15\text{ ms}$ ($t < 0.015\text{ s}$), generating a tonal acoustic thud dropping from $160\text{ Hz}$ to simulate beater impact against the head:
     $$\text{freq\_punch} = 160.0 \cdot \text{punch\_env}, \quad \text{punch\_signal} = \sin(\text{phase\_punch}) \cdot \text{punch\_env} \cdot \text{punch}$$

3. **Asymmetrical Console Saturation & Lo-Fi Bit-Crushing:**
   - **Mackie Saturation:** Adds a DC phase bias (`+0.05 * warmth`) and passes through `std::tanh` multiplied by pre-gain to impart a warm, woody vintage console analog grit.
   - **Akai S1000 Bit-Crush:** When `bit_depth < 15.9f`, samples are quantized to emulate vintage 12-to-16 bit DAC rounding:
     $$\text{sample} = \frac{\text{round}(\text{sample} \cdot \text{steps})}{\text{steps}}$$
   - A final $3\text{ ms}$ buffer fade-out (`apply_buffer_fadeout`) cleans up the tail.
