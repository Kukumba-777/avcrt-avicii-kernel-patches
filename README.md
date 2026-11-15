# Kernel Patches for OnePlus Nord (avicii / lito) — AVCRT & Build Fixes

A collection of patches for the Linux 4.19 kernel targeting the **OnePlus Nord** (`avicii`, Qualcomm Snapdragon 765G / `lito` platform). These patches fix common kernel build issues and introduce kernel-level audio controls required by the **AVCRT** real-time voice changer daemon (`avcrtd`).

---

## 📋 Patch Overview

This repository includes both a unified all-in-one patch and isolated modular patches:

| File | Description |
|---|---|
| **`avcrt-kernel-unified.patch`** | **Recommended (All-in-One)**: Combines all changes (AVCRT audio kcontrols + essential kernel build & camera PMIC fixes). |
| `avcrt-build-fixes.patch` | Build fixes only (rear camera PM8008 PMIC, DTC compiler flag, symbol export for external Wi-Fi drivers). |
| `avcrt-txmacro.patch` | Audio kcontrol only (`AVCRT Dec Mute` for clean mic silencing on cellular calls). |
| `avcrt-kernel.patch` | `AVCRT Dec Mute` + Device Tree compiler (DTC) build fix. |

---

## 🛠 Detailed Component Breakdown

### 1. Audio Subsystem for AVCRT (`techpack/audio`)
* **`AVCRT Dec Mute`** (`techpack/audio/asoc/codecs/bolero/tx-macro.c`):
  * Exposes an ALSA mixer control (`AVCRT Dec Mute`) taking a bitmask of TX decimators.
  * Directly forces the digital MUTE bit (`0x10`) in `BOLERO_CDC_TXn_TX_PATH_CTL`.
  * Allows the `avcrtd` daemon to mute modem-bound decimators at the codec level during cellular calls (GSM/VoLTE). This ensures the far end hears only the pitch-shifted audio injected via `incall_music` without the original voice leaking through. Default is `0` (off, stock behavior).
* **`AVCRT CVP Rebuild`** (`techpack/audio/asoc/msm-pcm-routing-v2.c`):
  * Provides surgical in-kernel vocproc teardown and rebuild (`voc_disable_device()` + `voc_enable_device()`) across active voice sessions without restarting `audioserver`.
  * Fixes the ADSP CVP latch bug when swapping audio accessories mid-call (e.g., 3.5mm analog jack to USB Type-C), avoiding USB soundcard re-enumeration flicker loops. Trigger auto-clears on write.

### 2. Kernel Build Fixes
* **Rear Camera PMIC Fix (`Makefile`)**:
  * Sets `export BRAND_SHOW_FLAG ?= oneplus`, forcing the build system to select `qcom_pm8008-regulator-nord` instead of the generic regulator driver. Without this, the PM8008 camera PMIC never reaches `VREG_READY` and rear cameras receive no power (black screen).
* **Device Tree Compiler Fix (`scripts/Makefile.lib`)**:
  * Appends `-f` to `cmd_dtc` so strict DTC checks do not fail on `lito-mtp-overlay.dtbo`. Enables successful compilation of `dtbs` and `Image.gz-dtb` targets.
* **External Wi-Fi Module Support (`kernel/cred.c`)**:
  * Adds `EXPORT_SYMBOL(get_task_cred)`, required for linking external out-of-tree `wlan.ko` drivers (such as NetHunter monitor mode / frame injection drivers). Harmless when WLAN driver is built-in (`=y`).

---

## 🚀 How to Apply

Navigate to the root directory of your kernel source tree:

```bash
cd /path/to/kernel/source
```

### Method 1: Using `git apply` (Recommended)

Check if the patch applies cleanly (dry run):
```bash
git apply --check /path/to/patches/avcrt-kernel-unified.patch
```

Apply the patch:
```bash
git apply /path/to/patches/avcrt-kernel-unified.patch
```

> **Tip:** To revert the patch:
> ```bash
> git apply -R /path/to/patches/avcrt-kernel-unified.patch
> ```

---

### Method 2: Using standard `patch`

Apply:
```bash
patch -p1 < /path/to/patches/avcrt-kernel-unified.patch
```

Revert:
```bash
patch -p1 -R < /path/to/patches/avcrt-kernel-unified.patch
```

---

## ⚙️ Mixer Controls Usage (On-Device)

Once the patched kernel is running, controls can be queried and set via root shell / ADB:

```bash
# Check current decimator mute mask
tinymix "AVCRT Dec Mute"

# Force-mute specific decimators (bitmask example)
tinymix "AVCRT Dec Mute" 1

# Trigger mid-call CVP vocproc rebuild (auto-clears to 0)
tinymix "AVCRT CVP Rebuild" 1
```
