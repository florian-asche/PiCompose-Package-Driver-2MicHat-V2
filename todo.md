# To do for PiCompose-Package-Driver-2MicHat-V2

Open points and work that was postponed. Remove an entry in the change that settles it.

1. **Test on hardware:** Install on Raspberry Pi OS Trixie with the HAT, reboot and check that the card `seeed2micvoicec` appears and the volume service sets the controls. Nothing was tested on hardware.

2. **Updating the overlay:** `src/respeaker-2mic-v2_0-overlay.dts` was copied from the `master` branch of Seeed-Studio/seeed-linux-dtoverlays on 2026-11-17. Check from time to time whether upstream changed it.
