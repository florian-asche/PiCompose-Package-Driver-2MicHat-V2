# To do for PiCompose-Package-Driver-2MicHat-V2

Open points and work that was postponed. Remove an entry in the change that settles it.

1. **Test on hardware:** Install on Raspberry Pi OS Trixie with the HAT, reboot and check that the card `seeed2micvoicec` appears and the audio works. Nothing was tested on hardware.

2. **Test the profile set on hardware:** Check that `pactl list cards` shows the card with the profile `output:stereo-fallback+input:stereo-fallback` and a sink, that `HP DAC` and `Line DAC` are at 100 % (0 dB) after a boot with the sink at 100 %, and that the sink volume changes them. The profile set was written from the files of PipeWire 1.4.2 and not tested. The element names (`HP DAC`, `Line DAC`) and merging both into one volume are assumptions. If the card is silent or missing, remove `89-picompose-driver-2michat-v2.rules`.

3. **Pin the download:** The overlay is built from the default branch of seeed-linux-dtoverlays at the time of the build, as in the image. A pinned commit would make builds repeatable. Decide whether that is wanted.
