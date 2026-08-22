# gauguin-kernel-builder-LOS

GitHub Actions workflow that builds a custom kernel for the Xiaomi Mi 10T Lite (**gauguin**) directly from the latest [LineageOS kernel source](https://github.com/LineageOS/android_kernel_xiaomi_gauguin) (`lineage-23.2`), with one patch applied automatically on every run.

This repo contains no kernel source of its own — it always builds against LineageOS's current upstream, so there's no fork to keep in sync or rebase.

## What it does

1. Checks out `LineageOS/android_kernel_xiaomi_gauguin` (`lineage-23.2`) fresh on every run.
2. Applies a one-line fix to `drivers/power/supply/qcom/qpnp-qg.c` (see below). The build **fails loudly** if the target line is no longer found, instead of silently skipping the patch — a signal that upstream changed and the patch needs a manual look.
3. Builds `Image.gz-dtb` and `dtbo.img` with Clang + the Android GCC 4.9 prebuilts.
4. Packages the result as a flashable AnyKernel3 zip.

## Included fix

**Battery percentage freeze with non-OEM batteries.**

Xiaomi's fuel gauge driver (`qpnp-qg.c`) tries to authenticate a 1-Wire (Maxim DS28E) chip present only in OEM batteries before loading the battery profile. With a non-OEM battery, authentication never succeeds, and profile loading gets stuck retrying instead of falling back to the generic resistance-based profile — freezing the reported battery percentage.

The fix forces `chip->batterysecret_support = false`, so all batteries (OEM or not) always use the standard profile lookup by battery ID resistance, skipping the OEM authentication check entirely.

## Usage

1. Go to the **Actions** tab → **Build gauguin kernel** → **Run workflow**.
2. Once finished, download the `Gauguin-Kernel-AnyKernel3` artifact.
3. Flash the zip from recovery.

## Notes

- Target device: Xiaomi Mi 10T Lite (gauguin / gauguinpro), running LineageOS 23.2.
- No slot suffix on `boot` (non-A/B device) — `BLOCK`/`IS_SLOT_DEVICE` are set accordingly in the AnyKernel3 template.
- Ramdisk is repacked with `RAMDISK_COMPRESSION=gzip` to avoid a chunk-size miscalculation bug in `magiskboot`'s `lz4_legacy` repacking for this device, which otherwise corrupts the ramdisk and causes a boot-time kernel panic.
