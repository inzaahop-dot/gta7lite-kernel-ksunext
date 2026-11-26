# C6 — Samsung OSRC source-archive audit (release `13155`, SM-T225 LTE / `T225XXSBEYE4`)

Audit date: 2026-11-26. Scope: read-only audit of official Samsung Open Source Release Center (OSRC) metadata and source-archive **listings/text only**, compared against local upstream commit `4497b6897b9d97053bcd99c2551ef2356cb69278`. No firmware, boot, recovery, or device binary was downloaded; no archive content was executed; no build, patch, flash, unlock, wipe, reboot, or device command was performed. The only repository file written by this cell is this report.

## 0. Evidence classes and access limitation

| Class | Meaning |
|---|---|
| **A** — authoritative OSRC metadata | Live OSRC HTML retrieved with `webfetch` from `https://opensource.samsung.com/uploadSearch?...`. |
| **M** — mirror of named OSRC archive | Internet Archive item `samsung-opensource-13155`, files with the exact OSRC archive names and matching MD5/SHA-1 recorded in the item metadata. Third-party host; not a Samsung-authenticated download. |
| **L** — local pinned tree | Clean checkout at `/tmp/opencode/gta7lite-kernel-src`, HEAD `4497b6897b9d97053bcd99c2551ef2356cb69278`, `kernel: sync T220XXS9EYA4`. |
| **I** — inference | Explicitly labelled deduction; never used as the sole basis for a fact. |

**Direct OSRC download was not completed.** The OSRC row is public (A), but the download UI uses `POST /downSrcMPop` with a per-page CSRF header. In this environment, direct/modal endpoints returned Cloudflare 403, HTTP 520, or the OSRC “Sorry! We're doing some work on site” page. No login credential was supplied or used. Archive bytes were therefore obtained only from the public Internet Archive mirror (M); see §2 and §8.

## 1. Release metadata — verified (A)

Two live OSRC queries succeeded during this audit:

- `https://opensource.samsung.com/uploadSearch?searchValue=T225XXSBEYE4` → **1 result**.
- `https://opensource.samsung.com/uploadSearch?searchValue=SM-T220` → **3 results**.

The first row of both pages contains release/upload id **`13155`** and announcement attachment id **`39127`**:

| Field | Value from OSRC HTML |
|---|---|
| Classification | `Mobile Phone` |
| Models | `SM-T220`, `SM-T225`, `SM-T225N`, `SM-T227`, `SM-T227U` |
| Versions | `T220XXSAEYE4`, `T225XXSBEYE1`, `T227USQSCEYE2`, `T227UVLSCFYE2`, `T225NDXSAEYE4`, `T225NKOSAEYE4`, **`T225XXSBEYE4`**, `T227JXSAEYE4`, `T225XXUBEZB1` |
| Source archive names | **`SM-T220_14_Opensource.zip`**, **`SM-T225_T225N_T227_14_Opensource_dts.zip`**, `SM-T227U_NA_14_Opensource_dts.zip` |
| JS source control | `showSrcDownPop('13155')` → `POST /downSrcMPop`, `uploadId=13155` |
| JS announcement control | `showAnnDownPop('13155','39127')` → `POST /downAnnMPop`, `uploadId=13155`, `attachId=39127` |

This confirms that Samsung associates the owner's exact build `T225XXSBEYE4` with a **two-archive T225 source package**: a T220 full kernel archive plus a small T225-family `_dts.zip` delta. The OSRC row itself does not disclose archive contents.

## 2. Archive acquisition, exact paths, and hashes (M)

Only the two named source archives from release `13155` were fetched, into `/tmp/opencode/osrc-13155/` (outside the repository):

| Archive | Mirror URL used | Size (bytes) | MD5 | SHA-1 | SHA-256 |
|---|---|---:|---|---|---|
| `SM-T225_T225N_T227_14_Opensource_dts.zip` | `https://archive.org/download/samsung-opensource-13155/SM-T225_T225N_T227_14_Opensource_dts.zip` | 43,156 | `2833090e140962c4dc51ba1d63cd0cc6` | `8eb53fb63e09c0cd4c3ad069689074ae21a4a33e` | `a4597bf68e43a6c9462ac775f2980b39d8cb6b4fd95b6bc99ef87f9c29ab3634` |
| `SM-T220_14_Opensource.zip` | `https://archive.org/download/samsung-opensource-13155/SM-T220_14_Opensource.zip` | 243,745,320 | `f4f983325f42cfe0ee6520d9dc143291` | `945900195675178a9d63d084d2c49da9005bdea2` | `5bed82eb953cd6b726a6776e161bac1f904ec8c45afdb42ed2ac3648aaa9e806` |

Internet Archive metadata: `https://archive.org/metadata/samsung-opensource-13155`; item title `samsung-opensource-13155`, creator `Samsung`, collection `samsung-opensource`, `publicdate 2025-06-16`. The MD5/SHA-1 values above match that metadata exactly. SHA-256 was computed locally from the downloaded bytes. No Samsung-provided checksum was retrievable, so these are **self-consistent mirror hashes**, not an authenticated Samsung signature.

## 3. Archive manifests (text listing only)

### 3.1 `SM-T225_T225N_T227_14_Opensource_dts.zip`

`unzip -l` shows exactly two members:

```text
    42405  2026-08-10 07:54   T225XXSBEYE4_kernel.tar.gz
     1961  2026-08-10 07:54   T225XXSBEYE4_kernel.txt
---------                     -------
    44366                     2 files
```

The inner tar contains only four regular files after directory entries:

```text
Kernel/arch/arm64/boot/dts/mediatek/Makefile
Kernel/arch/arm64/configs/gta7lite_defconfig
Kernel/arch/arm64/configs/gta7lite_00_defconfig
```

**There is no `.dts` or `.dtsi` file in the T225 delta archive.** Despite the archive name, it is a build/config delta, not an overlay source drop.

`T225XXSBEYE4_kernel.txt` says, verbatim in its first two steps:

```text
1. Download and unzip the kernel source of T220XXSAEYE4.

2. Unzip and update the kernel source of T225XXSBEYE4.
```

It then gives the Android 14 build command using `gta7lite_defconfig` and Clang `r383902` paths. The build instructions were read, not run.

### 3.2 `SM-T220_14_Opensource.zip`

`unzip -l` shows four members:

```text
212170511  2025-06-16 10:48   Kernel.tar.gz
 33300288  2025-06-16 10:48   Platform.tar.gz
     1849  2025-06-16 10:48   README_Kernel.txt
     2061  2025-06-16 10:48   README_Platform.txt
---------                     -------
245474709                     4 files
```

A targeted `tar -tzf` of `Kernel.tar.gz` confirmed the LTE/OT8 DTS texts used below:

```text
./arch/arm64/boot/dts/mediatek/mt8768t_gta7lite_eur_open_00.dts
./arch/arm64/boot/dts/mediatek/mt8768t_gta7lite_usa_single_00.dts
./arch/arm64/boot/dts/mediatek/mt8768t_gta7litewifi_eur_open_00.dts
./arch/arm64/boot/dts/mediatek/mt8768t-gta7lite_common.dtsi
./arch/arm64/boot/dts/mediatek/ot8.dts
./arch/arm64/boot/dts/mediatek/Makefile
./arch/arm64/boot/dts/samsung/mt8768t_gta7lite_eur_open_00.dts
./arch/arm64/boot/dts/samsung/Makefile
./arch/arm64/configs/gta7litewifi_defconfig
./arch/arm64/configs/gta7litewifi_00_defconfig
```

Only the listed text files were extracted to `/tmp/opencode/osrc-13155/t220-text/` and `/tmp/opencode/osrc-13155/t225-text/`. `Platform.tar.gz` and all binaries were left unopened.

## 4. Text comparison with pinned commit `4497b689…` (L vs M)

### 4.1 DTS / DTBO evidence

| File | Pinned `4497b689…` | OSRC `13155` BEYE4-derived text | Result |
|---|---|---|---|
| `mediatek/mt8768t_gta7lite_eur_open_00.dts` | `#include "ot8.dts"`; SHA-256 `592ff044889e8768e4c5d18f4149ed735a3b919299415b01d6f25c64e1fbc020` | `#include "../mediatek/ot8.dts"` (from T220 base); SHA-256 `4cefdb4cb6844eb8da411e7ff8c42222f9284cc10a92eb05bb6a4950487130f2` | **One-line mismatch.** Same compatible string `Samsung,Tab-A7-Lite EUR OPEN 00`, same `dtbo-hw_rev` range `0x0–0x20`. |
| `samsung/mt8768t_gta7lite_eur_open_00.dts` | identical to T220 archive copy (`diff -u` clean) | identical to pinned | Match. |
| `mediatek/mt8768t-gta7lite_common.dtsi` | identical to T220 archive copy (`diff -u` clean) | identical to pinned | Match. |
| DTS Makefile | config-driven: builds `.dtb` + `.dtbo` from `CONFIG_BUILD_ARM64_*_IMAGE_NAMES` | pinned tree: conditional overlay logic; OSRC T225 delta replaces it with hardcoded `dtb-y += mt6765.dtb` and `dtb-y += mt8768t_gta7lite_eur_open_00.dtb`, no `dtbo-y`; SHA-256 of T225 Makefile `9a09e1c23535da361d32a1c533f823b992d04ed94bd77e3f8117b291f87d56b9` | **Material build-path difference**, not just a source-string difference. |

The T220 base archive's own DTS Makefile selects the **Wi‑Fi** DTB (`mt8768t_gta7litewifi_eur_open_00.dtb`), while the T225 delta overwrites it to select the **LTE** DTB (`mt8768t_gta7lite_eur_open_00.dtb`). This is exactly the “base + delta” model described in `T225XXSBEYE4_kernel.txt`.

### 4.2 Defconfig evidence

| File | Pinned `4497b689…` | OSRC `13155` BEYE4-derived text | Result |
|---|---|---|---|
| `arch/arm64/configs/gta7lite_defconfig` | SHA-256 `ec58ddc2892b881998c1d82b7763c2e50fcbe1a8f86293fa6c862922fc8f6492`; header says `gcc (GCC) 14.1.1 20240720` | SHA-256 `f5bd65e8a5afd0aafefcd66923f13ab752c4b531e8a34ad3e57017f735adf984`; header says `Android (6443078 based on r383902) clang version 11.0.1`; 21 added / 11 removed lines vs pinned | **Not byte-identical.** The pinned config appears locally regenerated with GCC 14, not the Samsung Clang 11 config. |
| `arch/arm64/configs/gta7lite_00_defconfig` | **absent** from the pinned tree | SHA-256 `b5dad4d7aaee6451033e55c98c0894834efcef87a7d8f0f82e97594571977615`; 19,902-byte fragment-like config | New file introduced by the T225 delta; no pinned counterpart. |

Complete changed-line set in `gta7lite_defconfig` (pinned → OSRC T225):

```text
-# Compiler: gcc (GCC) 14.1.1 20240720
+# Compiler: Android (6443078 based on r383902) clang version 11.0.1 (...)
-CONFIG_CC_IS_GCC=y
-CONFIG_GCC_VERSION=140101
-CONFIG_CLANG_VERSION=0
+CONFIG_GCC_VERSION=0
+CONFIG_CC_IS_CLANG=y
+CONFIG_LD_IS_LLD=y
+CONFIG_CLANG_VERSION=110001
+CONFIG_TOOLS_SUPPORT_RELR=y
+CONFIG_CC_HAVE_SHADOW_CALL_STACK=y
-CONFIG_BUILD_ARM64_APPENDED_DTB_IMAGE=y
-CONFIG_BUILD_ARM64_DTB_OVERLAY_IMAGE=y
-CONFIG_BUILD_ARM64_APPENDED_DTB_IMAGE_NAMES="mediatek/mt6765"
-CONFIG_BUILD_ARM64_DTB_OVERLAY_IMAGE_NAMES="mediatek/mt8768t_gta7lite_eur_open_00"
+# CONFIG_LTO_CLANG is not set
+CONFIG_ARCH_SUPPORTS_SHADOW_CALL_STACK=y
+# CONFIG_SHADOW_CALL_STACK is not set
-CONFIG_PLUGIN_HOSTCC=""
+# CONFIG_TOUCHSCREEN_HX83108B_CHIPSET_O8 is not set
+# CONFIG_TOUCHSCREEN_HIMAX_IC_HX83102J is not set
+# CONFIG_ODM_SEC_COMMON is not set
-# CONFIG_TOUCHSCREEN_HIMAX_IC_HX83102J is not set
+CONFIG_BATT_TIME_TO_FULL=y
+# CONFIG_GXY_HS04_BATT_TIME_TO_FULL is not set
+CONFIG_F2FS_SEC_ENC_STRICT_MODE=y
+CONFIG_SECURITY_DEFEX=y
+# CONFIG_DEFEX_KERNEL_ONLY is not set
+# CONFIG_SECURITY_DEFEX_USER is not set
-CONFIG_KASAN_STACK=1
+CONFIG_KASAN_STACK=0
+# CONFIG_HQ_PROJECT_O8 is not set
```

Both configs retain `Linux/arm64 4.19.191`, `CONFIG_MACH_MT6765=y`, `CONFIG_MTK_PLATFORM="mt6765"`, `CONFIG_ARCH_MTK_PROJECT="ot8"`, `CONFIG_MTK_CCCI_DEVICES=y`, and `CONFIG_MTK_MD1_SUPPORT=10`. Neither contains `T225`, `BEYE4`, `SKZ`, or `Kazakhstan` as file content; the build id appears only in the archive member names.

### 4.3 Top-level Makefile evidence

`Makefile` in the T220 base archive differs from the pinned tree by 4 added / 4 removed lines:

```text
-ARCH		?= $(SUBARCH)
+ARCH            ?= arm64
+CROSS_COMPILE   ?= $(srctree)/toolchain/clang/host/linux-x86/clang-r383902/bin/aarch64-linux-gnu-
-HOSTLDFLAGS	+= -fuse-ld=lld
-CC		= $(CROSS_COMPILE)gcc
+CC		= $(srctree)/toolchain/clang/host/linux-x86/clang-r383902/bin/clang
-KBUILD_CFLAGS   += -O3
+KBUILD_CFLAGS   += -O2
```

Both contain the same `HUAQIN_BUILD` block at lines 985–988. Both declare `VERSION 4`, `PATCHLEVEL 19`, `SUBLEVEL 191`.

## 5. Shared-baseline question — answered

**Yes, a shared OT8 baseline is confirmed; no, the pinned commit is not the BEYE4 source.**

Shared/confirmed:

- Same base kernel `4.19.191` and top-level Makefile version triple.
- Same `mt6765` / `ot8` platform identifiers and LTE modem config (`CCCI`, `MD1_SUPPORT=10`).
- Same `mt8768t-gta7lite_common.dtsi` and same `samsung/mt8768t_gta7lite_eur_open_00.dts`.
- Same `Samsung,Tab-A7-Lite EUR OPEN 00` LTE DT compatible string.
- Same KSU-relevant knobs (§6).

Not the same release:

- Pinned `4497b689…` is `T220XXS9EYA4` (2025-01-01 patch lineage); BEYE4 is `T225XXSBEYE4` (2025-05-01 patch), built from `T220XXSAEYE4` plus a T225 delta.
- File-level mismatches are proven for the mediatek LTE DTS include path, the DTS Makefile build logic, `gta7lite_defconfig`, the new `gta7lite_00_defconfig`, and the top-level Makefile toolchain defaults.
- A full source-tree diff was **not** performed; therefore this is a file-level, not byte-level-whole-tree, conclusion.

## 6. SKZ / Kazakhstan overlay evidence — negative

- T225 delta archive: **no DTS/DTSI files at all**; only a DTS `Makefile` selecting `mt8768t_gta7lite_eur_open_00.dtb`.
- T220 base archive DTS inventory: `mt8768t_gta7lite_eur_open_00.dts` (`EUR OPEN 00`), `mt8768t_gta7lite_usa_single_00.dts` (`USA SINGLE 00`), `mt8768t_gta7litewifi_eur_open_00.dts`, `ot8.dts`, `ot8_fem.dts`, `mt8768t-gta7lite_common.dtsi`, and `TabA7_Lite/ot8-*.dtsi`. No filename or text containing `SKZ`, `Kazakhstan`, `KOR`, or `Kaz`.
- Extracted text files contain no literal `SKZ`/`Kazakhstan` string (verified by `grep -rFl`).

**Conclusion:** release `13155` provides no SKZ-specific DTS overlay. The active region identity for an SKZ device remains unresolved by source inspection; `EUR OPEN 00` is the only LTE DTB named by both the pinned tree and the official BEYE4-derived Makefile. Any claim that SKZ uses a hidden archive overlay is unsupported (I: region is probably handled outside DTS, e.g. CSC/vendor, but this cell does not verify that).

## 7. Config/DTS differences affecting the KSU assessment

| Area | Pinned vs official BEYE4-derived text | KSU relevance |
|---|---|---|
| Kprobes | `CONFIG_KPROBES=y`, `CONFIG_HAVE_KPROBES=y` in both; `CONFIG_KPROBES_ON_FTRACE` absent in both | Core kprobe availability unchanged. A hook design must **not** assume ftrace-backed kprobes in either tree. |
| FTRACE / tracing | `CONFIG_FTRACE=y`, `CONFIG_FUNCTION_TRACER=y`, `CONFIG_KALLSYMS=y`, `CONFIG_DEBUG_INFO=y` in both; `CONFIG_FTRACE_SYSCALLS` not set in both | No new blocker observed for symbol/ftrace-based approaches; hook design remains C2's scope. |
| Modules | `CONFIG_MODULES=y`, `CONFIG_MODULE_UNLOAD=y`, `CONFIG_MODVERSIONS=y`, `# CONFIG_MODULE_SIG is not set` in both | Module loading posture unchanged. |
| Security | `CONFIG_SECURITY=y`, `CONFIG_SECURITY_SELINUX=y`, `CONFIG_SECURITY_SELINUX_DEVELOP=y` in both; official adds `CONFIG_SECURITY_DEFEX=y` | New Samsung DEFEX LSM surface in BEYE4; any KSU access-control interaction needs re-review. |
| DTBO/DTB build | Pinned config sets `CONFIG_BUILD_ARM64_DTB_OVERLAY_IMAGE=y` with `...OVERLAY_IMAGE_NAMES="mediatek/mt8768t_gta7lite_eur_open_00"`; official T225 config drops all `CONFIG_BUILD_ARM64_*` lines and the Makefile hardcodes plain `.dtb` targets with no `dtbo-y` | **Material for packaging:** do not carry over pinned DTBO-generation assumptions to BEYE4. Boot/DTBO layout must be re-derived before any artifact work. |
| Compiler/build | Pinned config regenerated with GCC 14 and `ARCH ?= $(SUBARCH)`; official uses Android Clang 11 and hardcodes `arm64` plus local toolchain paths and `-O2` | Build outputs will not be byte-comparable; the `-28847731-abT225XXSBEYE4` release string is still not produced by in-tree text (external build input remains likely, I). |
| Other security/peripheral deltas | Official adds `F2FS_SEC_ENC_STRICT_MODE`, `BATT_TIME_TO_FULL`, touchscreen changes, `KASAN_STACK=0`, clears `ODM_SEC_COMMON`/`HQ_PROJECT_O8` | Peripheral/security surface differs; not a direct KSU knob, but relevant to a full-tree assessment. |
| KSU code presence | No `KernelSU`/`ksu` string found in the inspected official text files | This audit does not establish KSU applicability; it only bounds the prerequisites. |

No conclusion is drawn here about hook placement, runtime behaviour, or boot acceptance.

## 8. Limitations and unresolved items

1. **No authenticated direct OSRC download.** OSRC metadata is authoritative (A), but archive bytes came from a third-party mirror (M). Direct `/downSrcMPop` POST could not be completed; GET/modal attempts returned 403/520 or the OSRC error page. MD5/SHA-1 match the mirror metadata, but no Samsung checksum/signature was available.
2. **No whole-tree diff.** Only manifests and selected text files (DTS, DTSI, Makefiles, defconfigs, READMEs) were inspected. A full source-vs-commit diff was outside this cell's text-only method.
3. **Inner tar timestamps are mirror/repack artifacts** (`2026-08-10`), not Samsung build dates; they were not used as build evidence.
4. **Announcement attachment `39127` was not opened**; only its OSRC id was recorded.
5. **No device inspection.** Actual runtime DTBO/DTB selection, CSC behaviour, and the source of the `uname` suffix remain unverified.
6. **Pinned tree is a point-in-time checkout**; `.github/workflows/source-preflight.yml` remains unpinned and was not changed (C3 ownership).

## 9. Verdict

**Exact release source identified at archive level; pinned commit is not the exact BEYE4 source; shared OT8 baseline confirmed; SKZ overlay unsupported.**

More specifically:

- Samsung OSRC release `13155` publicly ties `T225XXSBEYE4` to `SM-T220_14_Opensource.zip` + `SM-T225_T225N_T227_14_Opensource_dts.zip` (A).
- The T225 archive is a small delta whose own instructions require the `T220XXSAEYE4` full source as its base; it supplies only a DTS `Makefile` and two defconfigs (M).
- Compared to `4497b689…`, the official BEYE4-derived texts differ in DTS include path, DTS/DTBO build logic, defconfig (21 added / 11 removed lines), top-level Makefile, and include a new `gta7lite_00_defconfig` (M vs L).
- No SKZ/Kazakhstan DTS overlay exists in either inspected archive (M).
- KSU-relevant core config knobs are unchanged, but a new DEFEX option and a removed DTBO-generation path require reassessment (M vs L).

This strengthens C5's “plausible common baseline” to **confirmed shared baseline with proven non-identity of the inspected files**, while stopping short of any exact-whole-tree or runtime claim.

## 10. Boundary confirmation

No repository file other than `.mece/cells/C6-OSRC/REPORT.md` was created or modified. Previous reports, `SYNTHESIS.md`, `WAVE.md`, source, configs, and workflows were untouched. No build, patch, packaging, upload, firmware/boot/recovery/device binary download, archive execution, `adb`/`fastboot`/Odin command, flash, unlock, wipe, reboot, or partition operation was performed. No secret, credential, or CSRF token value was stored or reproduced. Temporary artifacts exist only under `/tmp/opencode/`.
