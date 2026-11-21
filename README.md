# PiCompose-Package-Driver-2MicHat-V2

[![Version](https://github.com/florian-asche/PiCompose-Package-Driver-2MicHat-V2/actions/workflows/version.yml/badge.svg?branch=main)](https://github.com/florian-asche/PiCompose-Package-Driver-2MicHat-V2/actions/workflows/version.yml) [![GitHub Release Version](https://img.shields.io/github/v/release/florian-asche/PiCompose-Package-Driver-2MicHat-V2?label=version)](https://github.com/florian-asche/PiCompose-Package-Driver-2MicHat-V2/releases) [![GitHub License](https://img.shields.io/github/license/florian-asche/PiCompose-Package-Driver-2MicHat-V2)](https://github.com/florian-asche/PiCompose-Package-Driver-2MicHat-V2/blob/main/LICENSE)

Package `picompose-driver-2michat-v2` for the [PiCompose APT repository](https://github.com/florian-asche/PiCompose-Repository). It is the driver for the [ReSpeaker 2-Mics Pi HAT V2.0](https://wiki.seeedstudio.com/respeaker_2_mics_pi_hat/), which has a TLV320AIC3104 codec (I2C address 0x18). Version 1 of the HAT has a different codec and is the package `picompose-driver-2michat-v1`. It is the stage `02-stage-audiodriver-2michat-v2` of the [PiCompose](https://github.com/florian-asche/PiCompose) image as a package.

## Requirements

The package is for Raspberry Pi OS only. It copies the overlay to `/boot/firmware/overlays` and changes the boot configuration of the Raspberry Pi (`config.txt`, `usercfg.txt`), which only has an effect on the Raspberry Pi firmware. The package does not check this when it is installed, so on any other system it installs but does nothing.

## Contents

| Part | Files |
| --- | --- |
| Device tree overlay | `/usr/lib/picompose-driver-2michat-v2/respeaker-2mic-v2_0.dtbo`, built while the package is built, see below. The `postinst` copies it to `/boot/firmware/overlays/respeaker-2mic-v2_0.dtbo` (or `/boot/overlays` if that is where the overlays are), the `postrm` removes the copy. |
| Boot configuration | The `postinst` enables `dtparam=i2c_arm=on` and adds `dtoverlay=respeaker-2mic-v2_0` to the boot configuration (`config.txt`, `usercfg.txt` if it exists). The `postrm` removes the overlay line again. Both are safe to run more than once. |
| Volume | A profile set for the sound card, so the volume of the sink controls `HP DAC` and `Line DAC` of the codec: `/usr/lib/udev/rules.d/89-picompose-driver-2michat-v2.rules` selects it, `/usr/share/alsa-card-profile/mixer/profile-sets/picompose-2michat-v2.conf` and `/usr/share/alsa-card-profile/mixer/paths/picompose-2michat-v2-output.conf` define it |

The kernel has the driver for the codec (`snd-soc-tlv320aic3x`), so no kernel module is built. After a reboot the card appears as `seeed2micvoicec`. The codec starts with `HP DAC` and `Line DAC` at -23.5 dB. The profile set makes WirePlumber (ACP) merge both into the volume of the sink, so a volume of 100 % puts both at 0 dB. The profile set takes effect when the card is detected, so after a reboot. It was not tested on hardware.

## Source of the overlay

The image clones the repository [seeed-linux-dtoverlays](https://github.com/Seeed-Studio/seeed-linux-dtoverlays) while it is built and runs `make overlays/rpi/respeaker-2mic-v2_0-overlay.dtbo`. The package build (`debian/rules`) does the same, so building needs network access, and it uses the state of the default branch at that time. No source is stored in this repository. The overlay is copied to `/boot/firmware/overlays`, where Raspberry Pi OS (Trixie) keeps them, and not installed there by dpkg: that directory is on a FAT partition without hard links, where dpkg cannot replace a file during an upgrade. The change takes effect after a reboot.

## One driver per system

All `picompose-driver-*` packages provide `picompose-audio-driver` and conflict with it. Installing one removes another, because a system has one audio device.

## Versioning and changelog

The version of the package is the first entry of `debian/changelog`, which is part of the package. You do not edit it by hand, the workflows write it.

1. **Pull request:** Every pull request needs exactly one of the labels `major`, `minor` and `patch`. Tick one box under "Version bump" in the description or set the label by hand. The check "Version label" fails until there is exactly one.
2. **Merge to `main`:** The workflow "Version" raises the version, adds the pull request titles to `debian/changelog`, commits it to `main`, creates the tag `v<version>` as pre-release and starts the build, which publishes to the suite `trixie-nightly`.
3. **Release:** Turn the pre-release into a release in the GitHub UI. That publishes the same version to the suite `trixie`.
4. **Any other branch:** `<version>+dev<timestamp>.<sha>`, kept as workflow artifact for 14 days, nothing is published.

The repository needs the secret `HUB_DISPATCH_TOKEN` and a branch protection of `main` that requires the check "Version label" and lets `github-actions[bot]` push, see the [DEVELOPMENT.md](https://github.com/florian-asche/PiCompose-Repository/blob/main/DEVELOPMENT.md) of the APT repository. The repository also has to be listed in `sources.yml` of the APT repository.

## Local build

```sh
sudo apt install build-essential debhelper devscripts fakeroot lintian
dpkg-buildpackage -us -uc -b
lintian ../picompose-driver-2michat-v2_*.deb
```

## License

This project is released under the [MIT License](LICENSE).
