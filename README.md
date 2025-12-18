# Drumlabooh kits

Free drum kits for Drumlabooh (LV2/VST3) and Drumrox (LV2) drum machines. Both plugins support a simple text-based kit format, and Drumlabooh also supports more complex XML-based kits.

These kits are usually installed via the Drumlabooh installer, AUR package, or by building from source with CMake and Make.

But if you downloaded just the plugin binaries, you need to install drum kits manually. Each kit is a directory with subdirectories, samples, a kit picture, and a text or XML kit definition file. On the [Release page](https://github.com/psemiletov/drum_sklad/releases) you can download all kits as a single ZIP archive. Choose the latest release there. Or download the [current snapshot](https://github.com/psemiletov/drum_sklad/archive/refs/heads/main.zip) of the repository.

Then, unpack kits. You will have the folder ```drum_sklad-VERSION``` with subdirectories (drum kits). Copy them (not the top-level ```drum_sklad-VERSION```) to one of the following directories:

```
$HOME/drum_sklad
$HOME/drumlabooh-kits
$HOME/drumrox-kits
/usr/share/drumlabooh-kits
/usr/share/drumrox-kits
```

More kits (possibly not Public Domain) you can find here - [Drumlabooh kits at Telegram](https://t.me/drum_sklad)

## Drumkit formats specification 1.0

Drumlabooh has two native formats: a text-based one (a quick and simple way to create a kit) and an XML-based one (for more complex kits).


## Text-based drumkit format

The text-based Drumlabooh kit format is very simple. A drum kit is a directory containing samples (WAV, AIFF, OGG, FLAC, MP3), an optional image, and a text file. The optional file is ```image.png``` or ```image.jpg```, which holds an image of the drum kit. The mandatory file is ```drumkit.txt``` with lines such as ```instrument name=filename.wav```. For example:

```
kick=kick.wav
snare=share.wav
hihat close=hhc.wav
```

For multi-layered samples, simply separate the file names with a comma, using the order from the "quiet" sample to the "loudest" one (multi-layered samples are a set of samples that differ in timbre, not just in volume):


```
kick=kick01.wav,kick02.wav,kick03.wav,kick04.wav
snare=share01.wav,share02.wav,share03.wav
hihat opened=hihat01.wav,hihat02.wav
```

Sample files can be organized into subdirectories.  For example, we put samples into the **Kick** and **Snare** directories.:

```
kick=Kick/kick01.wav,kick02.wav,
snare=Snare/share01.wav,share02.wav
```

To simplify kit creation, you can generate the file list using the following console command:

```
ls > snares.txt
```

Then use it as follows:

```
kick=Kick/kick01.wav,kick02.wav,
snare=Snare/snares.txt
```

Drumlabooh supports kits with **Round Robin** and **Random Order** layer modes, which must be set at the sample definition level in the drumkit file.  
**Round Robin** means that layers play one by one with each note hit, then start again from the first layer once the list ends. **Random Order** means that a layer is chosen randomly with each note hit.

To define a **Round Robin** instrument, use the **greater-than sign (`>`)** at the beginning of the sample name:


```
>kick=kick01.wav,kick02.wav,kick03.wav
```

To define a **Random Order** instrument, use the **asterisk (`*`)** at the beginning of the sample name:

```
*kick=kick01.wav,kick02.wav,kick03.wav
```

There is another multi-layer sampling mode. For example:

```
^kick=kick01.wav,kick02.wav,kick03.wav
```

This means that samples are handled as a normal multi-sample set (from quiet to loud), but the MIDI velocity value **does not affect** the output volume. Instead, MIDI velocity is used **only** to select which layer to play. This mode is outdated.

A drum kit can have a **built-in MIDI map**, which is used when Drumlabooh's MIDI map mode is set to **"Kit"**.  To assign, for example, MIDI note 36 to an instrument:

```
[36]kick=kick01.wav,kick02.wav,kick03.wav
```

## XML-based drumkit format

Complex drum kits are better developed using **XML**. Moreover, some Drumlabooh features can only be defined in **XML-based kits**. A drum kit can include both definition files - **text-based** and **XML-based**. In such case, the **XML-based file** takes priority and will be loaded.

The XML-based kit definition file is named ```drumkit.labooh```.

A typical such file looks like this:

```
<?xml version="1.0" encoding="UTF-8"?>
<root type="alt">
    
<sample name="Kick" note="36">
kick.txt
</sample>

<sample name="Snare 1" note="38">
snare.txt
</sample>

<sample name="Snare 2" note="40">
snare.txt
</sample>

<sample name="Hihat Close" note="42">
choosy-house-Hihat05.wav,choosy-house-Hihat10.wav
</sample>

<sample name="Tom 1" note="45">
tom.txt
</sample>

<sample name="Hihat Open" note="46">
open-hihat.txt
</sample>


</root>
```

Each sample instrument is defined by a `<sample>` element containing parameters and file names.

As you can see, we can use **direct file names** (e.g., WAV samples) or **files containing lists of file names** (e.g., `somelist.txt`).

Each sample instrument element (`<sample>`) can have the following parameters:

`name` - this name will appear on the **slot label**.

`note` - The MIDI note to map to this instrument. Used when the **MIDI map mode** option (in the GUI) is set to **Kit**.

`mute_group` – the number of the mute group. A sample with a `mute_group` number mutes all other samples with the same `mute_group` number. Otherwise, by default, Drumlabooh uses automatic muting for hi-hats.

`layer_index_mode` – defines the layer selection mode (Round Robin, random, etc.). Possible values:  
- `rnd` (random order)  
- `robin` (cycled layers)  
- `no_velocity` (do not use MIDI velocity at sample level evaluation, so we use sample/track volume only)
- `alt` (alternative samples). Sample layers will be switched using the **Plus (`+`)** and **Minus (`-`)** buttons in the sample slot. A slot contains a **set of alternative samples**, not velocity layers. Otherwise, in other modes, each slot contains **multiple (or one) layers of the same instrument** (default behavior).


Peter Semiletov