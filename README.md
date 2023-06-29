# drumrox-kits
Drum kits for Drumrox LV2 drum machine

To use them, just put kits (as unpacked dirs) to ```$HOME/drumrox-kits``` or ```/usr/share/drumrox-kits```.

More kits (possibly not Public Domain) you can find here - [Drumrox kits at Telegram](https://t.me/drumrox_kits)


## Drumkit format

The Drumrox kit format is very simple. The drum kit is a directory with samples (in formats those supported by libsndfile) and some image and text files. The optional file is ```image.png``` or ```image.jpg``` that holds an image of the drum kit. The mandatory file is ```drumkit.txt``` with lines such as ```instrument name=filename.wav```. For example:

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
