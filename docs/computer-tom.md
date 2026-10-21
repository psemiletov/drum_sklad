# Computer Tom Generator Documentation

The `computer_tom` generator is an electronic tom-tom synthesizer styled after vintage analog machines and 80s minimal electronic music (Kraftwerk aesthetic). It combines a clean tonal sine wave body with a rapid exponential pitch drop and an optional high-frequency square wave transient click.

Call format:
```text
computer_tom(param1=val1, param2=val2, ...)
```

## Parameters Table

| Parameter | Type | Default | Range | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`freq`** | `float` | `150.0` | `50.0` — `600.0` | Base fundamental tuning frequency of the tom (in Hz). |
| **`decay`** | `float` | `0.35` | `0.05` — `2.0` | Total decay time and ring duration of the tom (in seconds). |
| **`click`** | `float` | `0.2` | `0.0` — `1.0` | Mix level and intensity of the initial square-wave stick impact click. |

---

## Architecture and Sonic Features

1. **Exponential Pitch Envelope:**
   The oscillator starts at a significantly higher pitch ($base\_freq + 1.5 \times base\_freq$) and quickly drops down to the fundamental tuning (`freq`) via an exponential curve (`std::exp(-t * 30.0f)`), perfectly modeling a tightly-tuned synthetic drum membrane.
2. **Sinusoidal Body:**
   The core resonance is generated using a pure sine wave multiplied by an exponential amplitude envelope (`env_factor`), delivering a clean, warm electronic thump.
3. **Transient Click Layer:**
   During the first 8 milliseconds (`t < 0.008f`), a high-frequency square wave (`base_freq * 3.0f`) is blended in, providing the characteristic wooden or plastic stick attack sound found in classic electronic percussion modules.
