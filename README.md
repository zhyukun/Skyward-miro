![Skyward Recovery](assets/hero.jpg)

# Skyward V0.3.1 For Miro

English | [简体中文](README.zh-CN.md)

> **miro fork** — a personal fork of [kirklin/skyward](https://github.com/kirklin/skyward) carrying the REDMI K80 Pro (`miro`) adaptation by zhyukun. The upstream project, Kirk Lin's copyright and `LICENSE` are kept as-is; only the adaptation changes are added here. The adaptation lives on the `miro` branch; `main` keeps the upstream baseline unchanged.

## Device

| Item | Value |
| --- | --- |
| Device | REDMI K80 Pro / POCO F7 Ultra, codename `miro`, CN 24122RKC7C |
| SoC | Snapdragon 8 Elite SM8750 (platform sun, soc_id 618) |
| Base | Android 17 / SDK 37, HyperOS 4.0.0.8, 16KB page, arm64-v8a |
| Layout | A/B, dedicated recovery partition; active slot `_a` |
| Build | 2026-11-15, release **Skyward V0.3.1 For Miro** |

> [!WARNING]
> ### Scope — read before flashing
> This recovery image has only been tested on a REDMI K80 Pro (`miro`) running **HyperOS 4.0.0.8 (Android 17)**. Other devices are not recommended for flashing. The same model on **HyperOS 3.X (Android 16)** is also **not** recommended — the Android version differs.

## Verified on device

- **Touch**: works after screen lock and waking from off (the touch setting is rewritten by a background thread on a 1 s timer); backup and restore work.
- **Data decryption**: cold start passes on the first try after entering the lock password.
- **WLAN**: peach_v2 driver; `wlan0` probes on demand when the WLAN page is opened.
- **Vibration motor**: reuses the 17 Ultra resonant frequency and strength parameters, verified working on this device's motor.
- **Other**: slot reading, persist mount, BootControl service, MTP / ADB.

## Download

Image: [skyward-0.3.1-miro-a17-zhyukun.img](https://github.com/zhyukun/Skyward-miro/releases/download/v1/skyward-0.3.1-miro-a17-zhyukun.img) · [Release page](https://github.com/zhyukun/Skyward-miro/releases/tag/v1)

```text
file    skyward-0.3.1-miro-a17-zhyukun.img
size    104,857,600 bytes (100 MiB)
SHA-256 9e7110209ba45a36897b0560f706300674b8cb4a598de47ed2a505a292a0fd18
```

## Flashing and rollback

Only flash the recovery partition:

```sh
fastboot devices
fastboot flash recovery_a skyward-0.3.1-miro-a17-zhyukun.img
fastboot reboot recovery
```

- Requires an unlocked bootloader.
- Roll back: fastboot-flash the stock recovery backup or a previous image.

## Known issues and open items

1. **Data decryption stability not fully proven**
   - Only a single cold-start pass; no multi-round comparison yet; the exact cause of an occasional failure has not been captured to disk.
   - The readiness race root cause is unconfirmed; the strongbox instance never comes up.
   - The exact enum name of error `-28` is unverified; the "decrypt now vs warm-up first" comparison has not been run.
   - Whether whole-device s2idle adds a touch factor has not been ruled out.
2. **Non-fatal WLAN warnings**: `hdd_config_data_stall_param` fail, bmiss sysfs node creation failure, 6GHz disabled, MLO MLD ID 0 exists. These are driver configuration/feature-cut warnings in the trimmed recovery; 2.4/5 GHz scan, association and networking work normally, only 6 GHz and MLO are off. `wlan0` only probes after the WLAN page is opened.
3. **Upstream changes not yet merged**: open PRs #33 (crypto walks all protectors), #29 (pack includes missing stock ramdisk files), #60 (WLAN firmware alternatives), #48 (myron A17 PIN decrypt), #43 (socrates decrypt compatibility).
4. **System version**: the `[stock]` config registers the OS4.0.3.0.XOMCNXM base; the released image is still built on the HyperOS 4.0.0.8 base. Repacking on 4.0.3.0 needs the recovery base re-dumped from the new full package and the checksum backfilled.
5. **Update mechanism**: the built-in update check is hardcoded to query the upstream `kirklin/skyward` latest release; this fork's v1 package is not picked up by it, and the update page reports no available build when the upstream release has no miro asset. Only a manual one-shot online query from the update page; the asset codename and SHA-256 are checked, with no independent GPG / signature verification.

## Disclaimer

As of the 2026-11-15 build, upstream Skyward has not published a recovery image for this device on HyperOS 4.0. If an update is detected, make sure the package supports the current device and Android version. The adapter zhyukun takes no responsibility for data loss, device damage, or any other consequence caused by flashing a wrong image.

---

## Installing Skyward

Skyward needs an unlocked bootloader. Download the zip for your phone from
[Releases](https://github.com/kirklin/skyward/releases), where each shows as the
phone's model and codename, such as Xiaomi 17 Ultra (nezha). Check it against
`SHA256SUMS`, unzip it, and flash the image inside to the recovery partition of
the slot the phone runs from, with the phone in fastboot mode:

```sh
unzip skyward-v0.3.1-nezha-OS4.0.4.0.XPACNXM.zip
fastboot flash recovery skyward-v0.3.1-nezha-OS4.0.4.0.XPACNXM.img
fastboot reboot recovery
```

Afterwards Skyward updates itself over Wi-Fi from its newest release, on the
Skyward update page in Settings; a newer image, unzipped, also installs from its own
Install page. With **Flash Skyward again** on, a system update or zip that writes
over the recovery partitions gets Skyward back in them.

adb in Skyward is a root shell that needs no authorization on the phone: anyone
with the phone and a cable can reach its storage while it is in recovery.

## How dual boot works

Skyward installs a second system beside the one in the phone's own partitions,
whether that is HyperOS or a custom ROM, and switches between them from **Dual boot**
on its home page. The second system's root is a file in the data partition that the
filesystem may neither move nor delete, read and written through the partition
itself; the boot images of both systems are kept in another such file. A switch
writes only the boot and init_boot partitions of the slot that starts next, and
keeps each step on /metadata first, so a switch cut off by a loss of power is
finished when Skyward next starts. vendor_boot, dtbo, recovery, vbmeta, the firmware
and super stay as they are, and the second system starts with the installed
system's vendor_boot and dtbo.

Before anything else writes the boot partitions, Skyward puts the installed system's
boot images back: an install, a slot switch, fastboot, a factory reset, and a backup
or restore of the boot partitions. The second system waits while a system update is
still to be merged, and backups of the data partition leave it out.

A second system installs from a zip on the Install page, as any other package does:

| File | What it holds |
| --- | --- |
| `system.toml` | `id`, `name`, `version` and `filesystem`; `init`, the program the root starts with, `/sbin/init` unless given; `minimum`, the least room the root takes, in bytes |
| `boot.img` | The kernel the bootloader starts |
| `rootfs.img` | The root filesystem's image, of the `filesystem` named |
| `init_boot.img` | Only for a system with a first ramdisk of its own |

A system without an init_boot image of its own starts from one Skyward makes. Its
program loads the installed system's first-stage kernel modules the way that
system's first stage loads them, with their soft dependencies and the settings the
kernel command line gives them, finds the root in the data partition, mounts it and
starts `init` there. If anything stops it, it brings Skyward up, as a failed start
of the installed system ends in the recovery: it leaves the reason in misc, unpacks
the ramdisk of the slot's recovery partition over its own, as the kernel unpacks it
when the bootloader starts the recovery, and hands over to Skyward's first program,
so Skyward comes up however the bootloader treats a restart. Skyward shows why, and
the second system still starts next until the user switches on Dual boot. A start
that stops moving restarts into the recovery instead, and a phone restarted before
the hand-over finds the same request in misc at its next start, which brings
Skyward up before anything else. Where Skyward comes up neither way, the phone stays
where it stopped, and never starts the same again and again. A system with a first
ramdisk of its own finds Skyward's program at `/skyward/map`, which lays the root
out as a block device and prints its node. Each switch to the second system writes
its init_boot image with the program of the Skyward that switches, so an updated
Skyward starts it with the new program.

A running system switches by writing `boot-recovery` with
`--switch_system=installed`, or `--switch_system=` and the second system's `id`, to
misc, and restarting into recovery. Where the bootloader starts the slot again, as
the 13 Ultra's can with a USB cable in, Skyward's program finds the request in misc
at the next start and brings Skyward up before anything else.

## License

[GPL-3.0-or-later](LICENSE)
