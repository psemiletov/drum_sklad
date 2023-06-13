# drumrox-kits
Drum kits for Drumrox LV2 drum machine

To use them, just put kits (as unpacked dirs) to ```$HOME/drumrox-kits``` or ```/usr/share/drumrox-kits```.

More kits (possibly not Public Domain) you can find here - [Drumrox kits at Telegram](https://t.me/drumrox_kits)


## Drumkit format

The Drumrox kit format is very simple. The drum kit is a directory with samples (in formats those supported by libsndfile) and some image and text files. The optional file is ```image.png``` that holds an image of drum kit. The mandatory file is ```drumkit.txt``` with lines such as ```instrument name=filename.wav```. For example:

```
kick=kick.wav
snare=share.wav
hihat close=hhc.wav
```
