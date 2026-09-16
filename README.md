# Taixingren M3 ImmortalWrt cloud build

[简体中文](README_CN.md) | English

This experimental GitHub Actions project builds ImmortalWrt for the
Taixingren M3 from the official immortalwrt/immortalwrt source. It defaults to
the v24.10.6 tag.

The M3 currently runs the Phicomm K2P image successfully, so this project keeps
the working K2P flash layout, Ethernet, Wi-Fi calibration, and LED definitions.
It keeps the phicomm,k2p device identity, enables the MT7621 native SD/MMC host
for the physical TF-card slot, and adds the required MMC and filesystem
packages to the existing K2P profile.

> [!WARNING]
> This is an inferred first-test port, not an officially supported ImmortalWrt
> target. Back up flash and confirm Breed or serial recovery before flashing.
> Run sysupgrade -T first and do not force an incompatible image with -F.

## Build with GitHub Actions

1. Push this repository to your GitHub account.
2. Open **Actions** and select **Build Taixingren M3 firmware**.
3. Click **Run workflow** and keep immortalwrt_ref at v24.10.6.
4. Optionally enter space-separated package names in extra_packages.
5. Download the completed artifact.

The artifact contains the M3 sysupgrade image, manifest, build metadata,
expanded diffconfig, and SHA-256 checksums. A GitHub Release is created only
when publish_release is selected.

## Package composition

ImmortalWrt adds its normal router defaults automatically, including the base
system, netifd, dnsmasq-full, firewall4, nftables, Dropbear SSH, LuCI, PPPoE,
IPv6, and wpad-openssl. The M3 device profile retains the K2P MT7615 firmware.

The TF-card additions are:

~~~text
kmod-mmc-mtk
block-mount
kmod-fs-ext4
kmod-fs-vfat
kmod-nls-utf8
~~~

Do not add kmod-sdhci-mt7620; it conflicts with kmod-mmc-mtk. Optional
maintenance tools can be supplied through extra_packages:

~~~text
e2fsprogs dosfstools fdisk
~~~

See [README_CN.md](README_CN.md) for repository layout, backup commands, and
pre-flash checks.

## Credits and license

The workflow is based on
[coachpo/immortalwrt-firmware-builder](https://github.com/coachpo/immortalwrt-firmware-builder).
Firmware source comes from
[immortalwrt/immortalwrt](https://github.com/immortalwrt/immortalwrt).
This builder repository uses the [MIT License](LICENSE); upstream components
retain their own licenses.
