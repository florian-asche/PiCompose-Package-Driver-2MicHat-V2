# To do for PiCompose-Package-Driver-2MicHat-V2

Open points and work that was postponed. Remove an entry in the change that settles it.

1. **Test on hardware:** Install on Raspberry Pi OS Trixie with the HAT, reboot and check that the card `seeed2micvoicec` appears and the volume service sets the controls. Nothing was tested on hardware.

2. **Pin the download:** The overlay is built from the default branch of seeed-linux-dtoverlays at the time of the build, as in the image. A pinned commit would make builds repeatable. Decide whether that is wanted.
