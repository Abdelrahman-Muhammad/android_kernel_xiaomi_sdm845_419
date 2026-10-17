# Poco F1 (beryllium) 4.19 Kernel — GitHub Actions build kit

## What this is
Builds a flashable AnyKernel3 zip for Poco F1 (beryllium) from:

- **Kernel:** `https://github.com/duckyduckG/android_kernel_xiaomi_sdm845_419` — branch `lineage-23.2_xiaomi`
  (this is the fork you must create and upload this kit into)
- **Config:** `vendor/sdm845-perf_defconfig` + `vendor/xiaomi/sdm845-common.config` + `vendor/xiaomi/beryllium.config`
  (exactly what the ROM's device tree uses: see `device/xiaomi/sdm845-common/BoardConfigCommon.mk` → `TARGET_KERNEL_CONFIG`)
- **Toolchain:** clang r487747c + GCC 4.9 (aarch64 + arm), `LLVM=1 LLVM_IAS=1`
  (this tree is clang-only: `TARGET_KERNEL_NO_GCC := true`, `CONFIG_LTO_CLANG=y`, `CONFIG_CFI_CLANG=y`)

## How to use
1. Fork `duckyduckG/android_kernel_xiaomi_sdm845_419` (default branch `lineage-23.2_xiaomi`).
2. Copy this kit into the **root** of your fork, keeping the structure:
   - `.github/workflows/build-kernel.yml`
   - `patches/0001-usb-typec-cc-bounce-debounce.patch`
   - `patches/0002-dsi-increase-cmd-transfer-timeout.patch`
3. GitHub → Actions → **Build Poco F1 (beryllium) Kernel 4.19** → Run workflow.
4. When done (~30–60 min), download artifact **PocoF1-4.19-Kernel** and flash the zip in recovery
   (it only touches the boot partition; device check = `beryllium`).

## What the patches do
- `0001-usb-typec-cc-bounce-debounce.patch` — `drivers/power/supply/qcom/smb-lib.c`:
  debounces Type-C CC bounce storms in `smblib_usb_typec_change()`. This stops the rapid
  "TypeC SINK insertion / TypeC removal" flapping from being forwarded to the USB psy / extcon / DWC3,
  which is what traps the controller (ADB/MTP dead after cable re-plug). Genuine plug/unplug is
  stable far longer than the 300 ms window, so real events are never lost.
- `0002-dsi-increase-cmd-transfer-timeout.patch` — `techpack/display/msm/dsi/dsi_ctrl.c`:
  raises `DSI_CTRL_TX_TO_MS` 200 → 1000 ms so occasional slow DSI command transfers don't error out.

Notes:
- The reboot/PS_HOLD warm-reset fix is **already included** in the `lineage-23.2_xiaomi` branch
  (`power: reset` commits, May 2026) — no patch needed for that.
- Both patches were produced and verified with `git apply --check` against the exact
  `lineage-23.2_xiaomi` files of this branch.
- Do NOT edit `drivers/power/supply/qcom/smblib.c` (doesn't exist) or `dwc3-qcom.c` (not built) —
  those paths from the Gemini guide are wrong for this tree.
