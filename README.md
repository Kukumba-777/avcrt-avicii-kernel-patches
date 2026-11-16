# OnePlus Nord 1 kernel patches (avicii / lito, vendor Linux 4.19)

Each patch has a single purpose. Overlapping combined patches have been removed
to prevent duplicate application. The patch order is listed in `series`.

## Patch contents

| File | Target | Purpose |
|---|---|---|
| `0001-audio-bolero-decimator-mute.patch` | `techpack/audio/asoc/codecs/bolero/tx-macro.c` | Adds `AVCRT Dec Mute`, a digital mute bitmask for Bolero decimators. AVCRT uses this to suppress the original microphone on compatible cellular routes. |
| `0002-audio-voice-cvp-recovery.patch` | `techpack/audio/asoc/msm-pcm-routing-v2.c` | Adds `AVCRT CVP Rebuild`, a vocproc disable/enable request without restarting audioserver. Reads return 0; writing 1 requests recovery; writing 0 does nothing. Requests are serialized, and q6voice errors are logged and returned to userspace. |
| `0003-avicii-oneplus-camera-pmic-selection.patch` | `Makefile` | Defaults `BRAND_SHOW_FLAG` to `oneplus` to select the OnePlus-specific PM8008 regulator. This is unrelated to voice processing. An explicit variable value can override the default. |
| `0004-build-dtc-force-output.patch` | `scripts/Makefile.lib` | Adds DTC `-f` to emit DTB output despite validation errors. This is a build workaround, not a device-tree repair; the output requires validation. It is not required by the DSP itself. |
| `0005-modules-export-get-task-cred.patch` | `kernel/cred.c` | Exports `get_task_cred` for external modules that reference this symbol, such as certain WLAN builds. AVCRT itself does not require this export. |

Control availability does not prove compatibility with every audio route. CVP
recovery requires on-device testing: successful requests for idle sessions may
be no-ops.

## Current source tree

All five changes are already present in the adjacent `NH-avcrt-avicii-kernel`
source tree, including the newly added CVP control. **Do not reapply this series
to that tree.** This is not a statement about the kernel running on a phone or
the result of hardware testing.

Do not build or flash automatically. The owner performs all builds and installation.

## Checking and applying patches

From the target kernel source root, inspect `git status` and `git diff` first.
Preserve existing user changes. For each selected patch:

```bash
git apply --check ../avcrt-avicii-kernel-patches/0002-audio-voice-cvp-recovery.patch
# Apply only if the changes are absent and the check succeeds:
git apply ../avcrt-avicii-kernel-patches/0002-audio-voice-cvp-recovery.patch
```

If the forward check fails, a reverse check can determine whether this version
of the change is already present. The following check does not revert anything:

```bash
git apply --reverse --check ../avcrt-avicii-kernel-patches/0002-audio-voice-cvp-recovery.patch
```

If both checks fail, investigate the diff and base version. Do not force application.

## Legacy filename mapping

| Previous file (removed or renamed) | Replacement |
|---|---|
| `avcrt-txmacro.patch` | `0001` |
| `avcrt-cvp-rebuild.patch` | `0002` |
| `avcrt-kernel.patch` | `0001` + `0004` |
| `avcrt-build-fixes.patch` | `0003` + `0004` + `0005` |
| `avcrt-kernel-unified.patch` | `0001`–`0005`, without a duplicate combined patch |

Use this mapping when reading older documents. Apply only missing changes, not
an old overlapping bundle on top of the new series.
