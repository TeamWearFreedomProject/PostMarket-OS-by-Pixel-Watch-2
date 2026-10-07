# PostMarket-OS-by-Pixel-Watch-2
I like Smartwatches.    uweeeeeeeeeeeeeeeeei

## Progress (2026-10-07)

**Status: Early postmarketOS port; not yet bootable.**

- [x] GitHub Actions installs pmbootstrap and builds an AArch64 test package.
- [x] Experimental `device-google-aurora` metadata package builds on Actions.
- [x] AArch64 **userspace-only** archive generated (kernel / initramfs absent).
- [ ] Kernel, matching modules, early initramfs, and safe rootfs boot process.
- [ ] postmarketOS on-device boot test.

[Passing CI: pmOS AArch64 build test #37563018624](https://github.com/TeamWearFreedomProject/PostMarket-OS-by-Pixel-Watch-2/actions/runs/37563018624)

[Passing CI: standalone AArch64 userspace archive #37564408562](https://github.com/TeamWearFreedomProject/PostMarket-OS-by-Pixel-Watch-2/actions/runs/37564408562) — roughly 524 MiB compressed tar within an artifact ZIP. **Not flashable.** `pmbootstrap install --no-image` still calls `mkinitfs`, which stops because we have not packaged an aurora Linux kernel; CI allows only that exact expected failure, validates the installed aarch64 userspace, locks archive account passwords and saves the offline files.

[Detailed porting notes and source credits](PORTING_NOTES.md)

### Why this starts from AsteroidOS and UBports

We have a **working AsteroidOS 2.2-nightly install on the Pixel Watch 2 Wi-Fi
(`aurora`)**. This project uses its tested Linux 5.15.144 boot/kernel/module
integration as a reference, alongside the Halium-13 / Ubuntu Touch Google
`google-eos` adaptation. It does **not** claim a working postmarketOS port.

- [Team Wear Freedom Project AsteroidOS build](https://github.com/TeamWearFreedomProject/AsteroidOS-Aurora-Build)
- [Upstream AsteroidOS aurora layer](https://github.com/AsteroidOS/meta-smartwatch/tree/master/meta-aurora)
- [UBports Google Pixel Watch 2 Android-13 adaptation](https://gitlab.com/ubports/porting/community-ports/android13/google-eos/google-eos)
- [Original aurora reverse-engineering notes](https://github.com/argosphil/aurora)

**No automatic flashing.** The working AsteroidOS rootfs is on shared
`userdata`, so neither switching A/B slots nor ordinary pmbootstrap
flashing protects it from an erase. Our experimental deviceinfo intentionally
sets `deviceinfo_flash_method="none"`. No output from the current CI is a
watch-ready boot or flash image.

As of September 2026, postmarketOS has been renamed **Nura**; the current development rootfs identifies as `ID=nura`. The project and tooling still use many `postmarketos-*` package names. [Official announcement](https://nura.eco/blog/2026/09/27/nura-rename/).
