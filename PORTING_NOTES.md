# Pixel Watch 2 Wi-Fi (aurora) — postmarketOS porting notes

Status: **early metadata and build-toolchain work only; no postmarketOS boot has been demonstrated.**

## Important: keep the working AsteroidOS installation

The watch currently boots AsteroidOS 2.2-nightly (Linux 5.15.144) from an
`asteroidos.ext4` loop file in a prepared ext4 **shared userdata** filesystem.
Its Linux shell, onboarding GUI, `/dev/fb0`, `/dev/dri/card0`, USB ADB, vendor mounts
and Halium-13 LXC start were observed on the physical watch (2026-10-07).
These observations establish a working reference, not PMOS support.

**Never run `fastboot flash userdata`, `fastboot -w`, `pmbootstrap flasher flash_rootfs`,
a super-partition resize, or a stock factory `flash-all` on this watch while
preserving that AsteroidOS rootfs.** A/B slots do not protect shared userdata.

The `port/device/testing/device-google-aurora/deviceinfo` deliberately uses
`deviceinfo_flash_method="none"`, and there are no production flashing scripts.
Everything under `port/` is experimental and must be treated as non-flashable.

## Source provenance and reusable facts

### 1. Proven AsteroidOS reference

- User build project:
  https://github.com/TeamWearFreedomProject/AsteroidOS-Aurora-Build
- Upstream aurora board integration:
  https://github.com/AsteroidOS/meta-smartwatch/tree/master/meta-aurora
- Aurora kernel recipe:
  https://github.com/AsteroidOS/meta-smartwatch/blob/master/meta-aurora/recipes-kernel/linux/linux-aurora_5.15.144.bb
- Android-style boot artifact recipe:
  https://github.com/AsteroidOS/meta-smartwatch/blob/master/meta-aurora/recipes-bsp/aurora-boot-images/aurora-boot-images.bb
- Working initramfs entry:
  https://github.com/AsteroidOS/meta-smartwatch/blob/master/meta-aurora/recipes-core/initrdscripts/initramfs-scripts-android/init.sh
- Stock vendor/B LXC integration:
  https://github.com/AsteroidOS/meta-smartwatch/blob/master/meta-aurora/recipes-android/aurora-vendor-mount/files/aurora-vendor-mount.sh

Critical observations from those files:

- Kernel **AArch64 Linux 5.15.144**. AsteroidOS uses an **armv7 userspace** for
  libhybris; postmarketOS should target **aarch64 userspace** instead.
- Kernel source is the UBports `kernel-for-google-eos` tree:
  `halium-13.0` at **`063840c5aae117bf0faac8b34fba0e37c9f619f8`** in the
  examined recipe. Treat this SHA as a known working starting point, not a
  guarantee it builds unchanged with postmarketOS tooling.
- Kernel configuration merges `gki_defconfig`,
  `arch/arm64/configs/vendor/monaco_GKI.config`, and
  `google-modules/soc/msm/sw5100.fragment`. It also depends on vendor modules.
- Boot format uses three **Android boot image v4** artifacts:
  `boot.img` (kernel), `init_boot.img` (early initramfs), and
  `vendor_kernel_boot.img` (selected kernel modules and DTB).
  Stock `vendor_boot` and Android 13 vendor components have separate roles.
- The bootloader/firmware situation is non-standard. Do **not** assume standard
  `pmbootstrap flasher` output is safe, and do not hardcode slot/partition
  edits in an unattended CI action.
- Existing AsteroidOS initramfs explicitly loads Qualcomm modules before
  mounting shared `/dev/mmcblk0p82` and loop-mounting
  `/asteroidos.ext4`. A default pmOS initramfs does *not* automatically
  reproduce that path.
- Existing vendor mount scripts use known slot-B `/super` dm-linear mappings
  and Halium Android LXC. These are **TWD9 / partition-layout-dependent**, and
  mapping them into pmOS requires an independent read-only verification.

### 2. Ubuntu Touch / Halium-13 reference

- Adaptation: https://gitlab.com/ubports/porting/community-ports/android13/google-eos/google-eos
- Kernel source: https://gitlab.com/ubports/porting/community-ports/android13/google-eos/kernel-for-google-eos
- The adaptation explicitly documents **TWD9.240405.001 Android 13** as a base;
  its install instructions subsequently *erase and resize logical partitions*.
  **Do not copy that flashing procedure** to this project.
- Useful inputs: ramdisk-overlay/module-loading logic, Android 13 HAL/LXC
  compatibility, and kernel/device adaptation. Reuse only with licensing,
  version, and actual hardware validation.

### 3. postmarketOS format

- Deviceinfo specification:
  https://docs.postmarketos.org/pmaports/main/deviceinfo-reference.html
- pmaports package sources: https://gitlab.postmarketos.org/postmarketOS/pmaports
- Target package location for future local pmaports overlay:
  `device/testing/device-google-aurora`.
- Initial metadata-only package contains **no `linux-google-aurora` dependency**.
  Adding a proper kernel package, modules, initramfs integration, and selecting
  a safe rootfs/boot process are explicit TODOs.
- The primary model is Pixel Watch 2 **Wi-Fi** `aurora`, not LTE `eos`.

## Development milestones (no device writes)

1. [x] Run pmbootstrap 3.x on GitHub Actions.
2. [x] Force-build a small **AArch64** package on Actions with QEMU `--no-cross`.
   Crossdirect on the Oct 7 runner failed to locate `liblto_plugin.so`; QEMU
   compilation succeeded.
3. [ ] Validate/build the experimental `device-google-aurora` metadata package.
4. [ ] Build an AArch64 userspace/rootfs as a standalone artifact with verified
   package architecture.
5. [ ] Port and package the pinned 5.15.144 kernel plus matching modules.
6. [ ] Construct an *offline-only* v4 boot/initramfs proof of concept,
   examine partition headers and exact byte sizes, and keep everything
   **UNVERIFIED / DO NOT FLASH**.
7. [ ] Design a separate and reversible boot experiment *without overwriting*
   existing AsteroidOS shared userdata or stock Android vendor partitions.
   Do not proceed to device flashing until a proven-safe approach exists.

## Observed/unknown distinctions

| Feature | AsteroidOS observed on real watch | postmarketOS |
| --- | --- | --- |
| Linux 5.15.144 AArch64 kernel | Yes | Not built yet |
| Armv7 Asteroid UI and onboarding | Yes | Not applicable |
| USB ADB root shell | Yes | Not verified |
| DRM `/dev/dri/card0` and `/dev/fb0` | Both exist | Not verified |
| Halium-13 LXC service | Runs; individual HALs still need testing | Not integrated |
| Wi-Fi AP scan | Yes | Not verified |
| Wi-Fi Internet via watch itself | Path not conclusively attributed | Not verified |
| Fastboot flashability | Three Asteroid images were flashed and GUI booted | **No boot image** |

**Credits:** Team Wear Freedom Project's on-device testing/build work; AsteroidOS
meta-smartwatch maintainers; UBports `google-eos` port maintainers; original
`argosphil/aurora` research; postmarketOS/pmbootstrap developers. Keep these
upstream source credits when reusing code.
