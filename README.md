# PiCompose-Package-Driver-2MicHat-V2

[![Version](https://github.com/florian-asche/PiCompose-Package-Driver-2MicHat-V2/actions/workflows/version.yml/badge.svg?branch=main)](https://github.com/florian-asche/PiCompose-Package-Driver-2MicHat-V2/actions/workflows/version.yml) [![GitHub Release Version](https://img.shields.io/github/v/release/florian-asche/PiCompose-Package-Driver-2MicHat-V2?label=version)](https://github.com/florian-asche/PiCompose-Package-Driver-2MicHat-V2/releases) [![GitHub License](https://img.shields.io/github/license/florian-asche/PiCompose-Package-Driver-2MicHat-V2)](https://github.com/florian-asche/PiCompose-Package-Driver-2MicHat-V2/blob/main/LICENSE)

Package `picompose-driver-2michat-v2` for the [PiCompose APT repository](https://github.com/florian-asche/PiCompose-Repository). It is the driver for the [ReSpeaker 2-Mics Pi HAT V2.0](https://wiki.seeedstudio.com/respeaker_2_mics_pi_hat/), which has a TLV320AIC3104 codec (I2C address 0x18). Version 1 of the HAT has a different codec and is the package `picompose-driver-2michat-v1`. It is the stage `02-stage-audiodriver-2michat-v2` of the [PiCompose](https://github.com/florian-asche/PiCompose) image as a package.

What differs from how the stage does it is documented in [diff.md](diff.md) (in German).

## Contents

| Part | Files |
| --- | --- |
| Device tree overlay | `/boot/firmware/overlays/respeaker-2mic-v2_0.dtbo`, built while the package is built, see below |
| Boot configuration | The `postinst` enables `dtparam=i2c_arm=on` and adds `dtoverlay=respeaker-2mic-v2_0` to the boot configuration (`config.txt`, `usercfg.txt` if it exists). The `postrm` removes the overlay line again. Both are safe to run more than once. |
| WirePlumber | `/etc/wireplumber/wireplumber.conf.d/51-disable-acp.conf` turns off the ALSA card profile handling (ACP) |
| Volume setup | `/usr/bin/configure_audio.sh`, `configure_audio.service` (enabled, not started at install) |

The kernel has the driver for the codec (`snd-soc-tlv320aic3x`), so no kernel module is built. After a reboot the card appears as `seeed2micvoicec`. The volume service waits for its `PCM` control, sets the outputs to 100 %, the input gain to 80 % and stores the ALSA state.

## Source of the overlay

The image clones the repository [seeed-linux-dtoverlays](https://github.com/Seeed-Studio/seeed-linux-dtoverlays) while it is built and runs `make overlays/rpi/respeaker-2mic-v2_0-overlay.dtbo`. The package build (`debian/rules`) does the same, so building needs network access, and it uses the state of the default branch at that time. No source is stored in this repository. The overlay is installed to `/boot/firmware/overlays`, where Raspberry Pi OS (Trixie) keeps them. The change takes effect after a reboot.

## One driver per system

All `picompose-driver-*` packages provide `picompose-audio-driver` and conflict with it. Installing one removes another, because a system has one audio device and all of them install `configure_audio.sh` and `configure_audio.service`.

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
