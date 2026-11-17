# Abweichungen von der Stage `02-stage-audiodriver-2michat-v2`

Dieses Dokument beschreibt, was das Paket `picompose-driver-2michat-v2` anders macht als die gleichnamige Stage im [PiCompose](https://github.com/florian-asche/PiCompose)-Repository. Verglichen wurde mit dem Stand von `main` (Commit `784abfb`, 17.11.2026). Gepflegt wird es auf Deutsch, auf Wunsch des Entwicklers.

Pflege: Jede Änderung, die eine Abweichung neu schafft, ändert oder beseitigt, aktualisiert dieses Dokument. Prüfe dabei gegen den aktuellen `main` von PiCompose.

## Anders gelöst

- **Das Overlay.** Die Stage klont `seeed-linux-dtoverlays` (`--depth 1`) beim Bau des Images und baut es mit `make overlays/rpi/respeaker-2mic-v2_0-overlay.dtbo`. Das Paket macht beim **Bau des Pakets** genau dasselbe (`debian/rules`, Netzwerkzugriff nötig, Stand des Standard-Branches zum Zeitpunkt des Baus, nicht festgelegt). Das Ergebnis liegt als `respeaker-2mic-v2_0.dtbo` im Paket. Der Quelltext liegt nicht im Repository.
- **Ort des Overlays.** Die Stage erkennt `/boot/firmware/overlays` oder `/boot/overlays` beim Bau. Das Paket installiert fest nach `/boot/firmware/overlays` (Raspberry Pi OS Trixie).
- **`config.txt`.** Die Stage erkennt die Datei (`/boot/config.txt`, `/boot/firmware/config.txt`, `usercfg.txt`) beim Bau und ändert sie. Das Paket macht dieselbe Erkennung in der `postinst`, mit demselben Ergebnis (`dtparam=i2c_arm=on` aktiv, `dtoverlay=respeaker-2mic-v2_0` vorhanden), jedoch mit zwei Unterschieden: Der Kommentar `#dtparam=i2c_arm=on` wird nur am Zeilenanfang aktiviert (die Stage ersetzt ihn an jeder Stelle), und beide Aufrufe sind beliebig oft wiederholbar. Die `postrm` entfernt die Zeile `dtoverlay=respeaker-2mic-v2_0` beim Entfernen des Pakets, die Stage kennt kein Entfernen.
- **Pakete der Stage** (`git`, `build-essential`, `device-tree-compiler`, `alsa-utils`): `git`, `ca-certificates` und `device-tree-compiler` sind `Build-Depends` (nur zum Bauen), `alsa-utils` ist `Depends`.
- Die Unit-Datei liegt in `/usr/lib/systemd/system/` statt in `/etc/systemd/system/`. Aktiviert wird sie durch `dh_installsystemd` im Paket (beim Installieren aktiviert, aber nicht gestartet) statt durch `systemctl enable` im Chroot der Stage. Der Inhalt der Unit ist unverändert.
- `51-disable-acp.conf` ist als Conffile gekennzeichnet.

## Zusätzlich im Paket

- `Depends` auf `wireplumber`.
- `Provides`, `Conflicts` und `Replaces` für `picompose-audio-driver`.

## Unverändert

- `51-disable-acp.conf`, `configure_audio.sh` und `configure_audio.service` sind Byte für Byte die Dateien der Stage.
