# drumrox-kits
Drum kits for Drumlabooh (LV2/VSTi) and Drumrox (LV2) drum machines. This repo shares kits because both plugins supports this simple format.

To such kits them, just put kits (as unpacked dirs) to (if Drumlabooh only installed) ```$HOME/drumlabooh-kits``` or ```/usr/share/drumlabooh-kits```, and
to (for Drumrox and Drumlabooh) ```$HOME/drumrox-kits``` or ```/usr/share/drumrox-kits```.

More kits (possibly not Public Domain) you can find here - [Drumlabooh kits at Telegram](https://t.me/drum_sklad)


## Drumkit format

The Drumrox/Drumlabooh kit format is very simple. The drum kit is a directory with samples (WAV, AIFF, OGG, FLAC, MP3) and some image and text files. The optional file is ```image.png``` or ```image.jpg``` that holds an image of the drum kit. The mandatory file is ```drumkit.txt``` with lines such as ```instrument name=filename.wav```. For example:

```
kick=kick.wav
snare=share.wav
hihat close=hhc.wav
```

For the multi-layered samples, just separate their file names with comma, using the order from "quiet" sample to the "loudest" one (multi-layered samples are the set of samples those differs with the timbre, not the volume):


```
kick=kick01.wav,kick02.wav,kick03.wav,kick04.wav
snare=share01.wav,share02.wav,share03.wav
hihat opened=hihat01.wav,hihat02.wav
```
