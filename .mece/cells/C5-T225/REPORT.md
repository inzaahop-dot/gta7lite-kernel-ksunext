# C5 — T225 target compatibility audit

Audit date: 2026-11-26. Scope: read-only provenance and compatibility research for the device evidence supplied by the owner. This cell does **not** design KernelSU Next hooks, change CI, or define a device test plan. No source archive, firmware, or binary was downloaded; no device command was run.

## 0. Evidence classes used

| Class | Meaning |
|---|---|
| **U** — user-supplied | Device-owner evidence quoted in the task (SM-T225 / `gta7lite`, MT6765, Android 14/API 34, fingerprint `samsung/gta7litexx/gta7lite:14/UP1A.231005.007/T225XXSBEYE4:user/release-keys`, kernel `4.19.191-28847731-abT225XXSBEYE4`, patch `2025-05-01`, sales/carrier code `SKZ`, read-only partition list). Treated as claim, not independently verified on a device. |
| **A** — authoritative | Samsung Open Source Release Center (OSRC), Samsung Notify Update pages, GitHub API/Web blob content at pinned commit. |
| **T** — third-party | SamFW, SamFrew, SamMobile. Corroboration only; not treated as proof of source identity. |
| **I** — inference | Explicitly labelled deduction; never used as sole evidence for a conclusion. |

## 1. Findings

### 1.1 The exact device build is tied to an authoritative Samsung source release — verified (A)

OSRC query `https://opensource.samsung.com/uploadSearch?searchValue=T225XXSBEYE4` returned **1 result**, upload/release id **`13155`** (announcement attachment id `39127`):

- Classification: `Mobile Phone`
- Models: `SM-T220`, `SM-T225`, `SM-T225N`, `SM-T227`, `SM-T227U`
- Versions (verbatim list): `T220XXSAEYE4`, `T225XXSBEYE1`, `T227USQSCEYE2`, `T227UVLSCFYE2`, `T225NDXSAEYE4`, `T225NKOSAEYE4`, **`T225XXSBEYE4`**, `T227JXSAEYE4`, `T225XXUBEZB1`
- Source archive names (verbatim): **`SM-T220_14_Opensource.zip`**, **`SM-T225_T225N_T227_14_Opensource_dts.zip`**, `SM-T227U_NA_14_Opensource_dts.zip`

Evidence: [OSRC exact-build search](https://opensource.samsung.com/uploadSearch?searchValue=T225XXSBEYE4); the same row and ids were observed in two independent retrievals of that URL. A later repeat query returned HTTP 520 (site-side error), so the row could not be re-fetched a third time; no archive was opened.

**What this establishes:** Samsung publicly associates the owner's exact build `T225XXSBEYE4` with source material. **What it does not establish:** that the archive named `SM-T220_14_Opensource.zip` is byte-identical to the GitHub tree examined below, or what files the `_dts.zip` contains. Those contents were **not inspected** (no download permitted in this wave). The `dts` suffix is an archive-name fact; inferring its contents is **I**.

### 1.2 Device build identity and region corroboration — verified (A/T) against user evidence (U)

| Item | Evidence | Class |
|---|---|---|
| `T225XXSBEYE4`, Android 14 (`U`), release date `2025-05-29`, security patch `2025-05-01` | [Samsung Notify Update, SM-T225](https://doc.samsungmobile.com/SM-T225/018843210521/mlt.html) | A |
| `SKZ` = Kazakhstan; CSC `T225OXMBEYE4`; bootloader/SW REV `B`; AP `T225XXSBEYE4` | [SamFW SKZ build page](https://samfw.com/firmware/SM-T225/SKZ/T225XXSBEYE4) | T |
| SKZ, build date `2025-05-13`, PDA `T225XXSBEYE4`, CSC `T225OXMBEYE4` | [SamFrew SKZ build page](https://samfrew.com/download/Galaxy__Tab__A7__Lite__/qjvr/SKZ/T225XXSBEYE4/T225OXMBEYE4) | T |

The owner's patch level (`2025-05-01`), fingerprint suffix, and `SKZ` sales/carrier code are consistent with these sources. Note: `SKZ` is **Kazakhstan**, not a Korean SK Telecom code (T). The read-only partition map in the owner evidence (`boot`, `recovery`, `dtbo`, `vbmeta`, `super`) is consistent with an A/B-less Samsung boot chain, but no partition operation was attempted.

### 1.3 Upstream commit `4497b689…` is a T220 January-patch sync, not the T225 May-patch build — verified (A)

| Property | Value / evidence |
|---|---|
| Commit | [`4497b6897b9d97053bcd99c2551ef2356cb69278`](https://github.com/gta7lite/android_kernel_samsung_ot8/commit/4497b6897b9d97053bcd99c2551ef2356cb69278) |
| Message | `kernel: sync T220XXS9EYA4` ([API](https://api.github.com/repos/gta7lite/android_kernel_samsung_ot8/commits/4497b6897b9d97053bcd99c2551ef2356cb69278)) |
| Author/commit date | 2025-05-18 (Zillion); unsigned; parent `3f6ea246eb39e4480936132f53665f86109ac082` |
| Tree | `9c16bcd9392fbed50d464920ea830e1655a4036a` |
| Actual diff | 7 added lines only: `HUAQIN_BUILD` block in `Makefile` (~985–987). The T220 sync was a long series of subtree commits; the tip commit is only the last one. |
| OSRC listing for T220 build | `SM-T220` / `T220XXS9EYA4` / `SM-T220_EUR_14_Opensource.zip` ([OSRC mobile list](https://opensource.samsung.com/uploadList?classification1=mobile_phone&menuItem=mobile)). An OSRC query for `T220XXS9EYA` returned 1 result with release id **`12636`** (single successful observation; later re-queries returned HTTP 520). |
| Samsung Notify for that T220 build | `T220XXS9EYA4`, Android 14, release `2025-02-12`, patch `2025-01-01`; `T220XXSAEYE4` is the later `2025-05-01` patch build ([SM-T220 notify](https://doc.samsungmobile.com/SM-T220/018911210526/wel.html)) |

Consequences:

1. The inspected tree is keyed to a **T220 (Wi-Fi model)** build with a **2025-01-01** patch, while the target is **T225 (LTE)** with a **2025-05-01** patch. Even the T220 entry inside the owner's release is `T220XXSAEYE4`, not `T220XXS9EYA4` — so the two are different release cohorts, not the same archive under a different name.
2. The repository does have genuine T225 ancestry: the 600 most-recent `master` commits sampled via the GitHub API include many `import T225XXU6DWL9` commits dated 2024-07-26, e.g. [`e5f6bdb50f516c959b8f64e778f092aa3f0cc70a`](https://github.com/gta7lite/android_kernel_samsung_ot8/commit/e5f6bdb50f516c959b8f64e778f092aa3f0cc70a) (`kernel: import T225XXU6DWL9`, files `Kconfig`, `MAINTAINERS`, `Makefile`, `kenv.mk`). OSRC also lists `T225XXU6DWL9` with `SM-T225_EUR_14_Opensource.zip` ([SM-T225 search](https://opensource.samsung.com/uploadSearch?searchValue=SM-T225)). This supports a shared OT8 baseline, but it is an **older** T225 build.
3. In the sampled 600 `master` commits and the first 100 commits of each of the 5 upstream branches, **no commit message contained `T225XXSBEYE4` or `BEYE4`** (GitHub commits API, paginated). Absence in commit messages is not proof of absence in content, but it removes an easy provenance shortcut.

### 1.4 LTE defconfig and DTS overlay identity vs device evidence — verified (A) + inference (I)

All upstream line references below are pinned to `4497b689…`. The local checkout has **no kernel source**; `.github/workflows/source-preflight.yml` checks out upstream `master` unpinned (C1 baseline, consistent with this audit).

| Comparison | Upstream value at `4497b689` | Owner evidence | Assessment |
|---|---|---|---|
| Base kernel version | `Makefile:2-5` = `4.19.191`; `arch/arm64/configs/gta7lite_defconfig:3` = `Linux/arm64 4.19.191` | `4.19.191-28847731-abT225XXSBEYE4` | Prefix matches; full `uname -r` suffix does not. |
| Local version suffix | `gta7lite_defconfig:22-23`: `CONFIG_LOCALVERSION=""`, `CONFIG_LOCALVERSION_AUTO=y`; [`scripts/setlocalversion`](https://raw.githubusercontent.com/gta7lite/android_kernel_samsung_ot8/4497b6897b9d97053bcd99c2551ef2356cb69278/scripts/setlocalversion) only emits git-derived suffix, or `-androidNN-KMI` when explicit args are supplied | `-28847731-abT225XXSBEYE4` | **Not reproducible from the tree as inspected.** Samsung build-time `LOCALVERSION`/`localversion*` input is unverified (**I**: likely injected outside the repo). |
| SoC/platform | `gta7lite_defconfig:287-291` `CONFIG_MACH_MT6765=y`; `:1514-1515` `CONFIG_MTK_PLATFORM="mt6765"`, `CONFIG_ARCH_MTK_PROJECT="ot8"` | MT6765 | Matches. |
| SoC name in DT | `mt8768t_gta7lite_eur_open_00.dts:14` compatible `Mediatek,MT8768T` | MT6765 | Equivalence of labels is **not defined in the inspected files** (**I**: likely same chip family). |
| LTE variant | `gta7lite_defconfig:1767-1768` `CONFIG_MTK_CCCI_DEVICES=y`, `CONFIG_MTK_MD1_SUPPORT=10`; DTBO at `:438-441` = `mediatek/mt8768t_gta7lite_eur_open_00` | SM-T225 LTE, `gta7lite` | Consistent with LTE family, but **no `T225`/`SM-T225` string in defconfig or DTS**. |
| Region / overlay | DTS `:2` "Generated by Samsung"; `:10` includes `mt8768t-gta7lite_common.dtsi`; `:14` `Samsung,Tab-A7-Lite EUR OPEN 00` | `SKZ` (Kazakhstan), CSC `T225OXMBEYE4` | **Mismatch of region identity**: `EUR OPEN 00` vs `SKZ`. |
| Available gta7lite DTS files | `arch/arm64/boot/dts/mediatek/` and `arch/arm64/boot/dts/samsung/` at `4497b689` contain only `mt8768t_gta7lite_eur_open_00.dts`, `mt8768t_gta7lite_usa_single_00.dts`, `mt8768t_gta7litewifi_eur_open_00.dts`, plus common dtsi/ot8 files ([GitHub contents API](https://api.github.com/repos/gta7lite/android_kernel_samsung_ot8/contents/arch/arm64/boot/dts/mediatek?ref=4497b6897b9d97053bcd99c2551ef2356cb69278)) | SKZ region | **No SKZ/KOR T225 DTS overlay found in the inspected upstream directories.** |
| Android generation | OSRC `14` archives and Samsung Notify Android 14 for both builds; tree contains `build.config.mtk.aarch64.tiramisu` | Android 14 / API 34 | Consistent at release level; tree-only clues remain indirect. |
| Bootloader/AVB | Not represented in defconfig/DTS evidence; owner reports bit `B` (T) and `vbmeta`/`dtbo` partitions (U) | — | Unknown from this cell. |

### 1.5 Differences that could affect KernelSU Next integration (constraints, not a hook design)

1. **Region DTBO divergence (highest risk).** Current `master` selects `mt8768t_gta7lite_eur_open_00` (`EUR OPEN 00`) while the device reports `SKZ`. The owner's OSRC release explicitly lists a T225-family `_dts.zip`; a region-correct DTS/DTBO source must be identified before any boot image is considered. (C2 owns hook design; C4 owns validation.)
2. **Base-build delta.** `T220XXS9EYA4` (2025-01-01 patch, synced 2025-05-18 upstream) vs `T225XXSBEYE4` (2025-05-01 patch, released 2025-05-29): four months of vendor security/kernel deltas are unaccounted for. Hook context and vendor hooks may differ even if the architecture matches.
3. **Release-string mismatch.** `4.19.191` matches, but the exact `uname` suffix is not derived from repo contents; any build based on `master` would need its own version provenance check before claiming equivalence to the stock boot image.
4. **Model/variant ambiguity.** `MT8768T` appears in DT compatible strings, `MT6765` in Kconfig. LTE and Wi-Fi share SoC strings; only modem config and overlay names separate them. Filename-only matching is unsafe.
5. **Unpinned source ref.** CI tracks `master` (C1). The commit assessed here is a point-in-time observation, not a reproducible input; a change of branch head changes every conclusion in §1.3–1.4.
6. **Vendor security surface.** Baseline config carries Samsung/MediaTek/KNOX/TEEGRIS options (C1 inventory). Whether the exact BEYE4 tree carries the same options is unverified; this cell draws no KSU compatibility conclusion from them.

## 2. Verdict

**Plausible common baseline — not an exact source match demonstrated; not merely "unverified".**

Rationale:

- **Verified:** the owner's exact firmware build is published by Samsung in OSRC release `13155` together with named source archives (§1.1), and Samsung's own update record confirms Android 14 / `2025-05-01` patch (§1.2). The device identity is therefore well grounded.
- **Verified:** the GitHub tree at `4497b689…` is a **T220 `T220XXS9EYA4`** sync with different patch lineage and a `EUR OPEN 00` LTE overlay (§1.3–1.4). It is a plausible **common OT8 baseline** (the repo also contains imported T225 `T225XXU6DWL9` source), but it is **not the T225 BEYE4 release**.
- **Unverified:** no byte-level or file-level comparison between OSRC release `13155` archives and `4497b689…` was performed (archives were not downloaded). No SKZ DTS overlay was found in the inspected upstream directories. No commit naming `T225XXSBEYE4` was found in the sampled history.

Therefore an "exact source match" claim would be unsupported. A bare "unverified" verdict would discard the authoritative firmware-to-release tie and the demonstrated shared OT8 ancestry; the accurate middle verdict is **plausible common baseline**.

## 3. Assumptions and unknowns

- **Assumption:** the owner's quoted `ro.*` and fingerprint values accurately describe the physical device; no device was queried to confirm them.
- **Assumption:** `T225XXSBEYE4` in OSRC row `13155` refers to the same PDA build as the owner's fingerprint; strongly supported by exact string equality but not cross-checked against a device-read build property in this wave.
- **Unknown:** contents and hashes of `SM-T220_14_Opensource.zip` and `SM-T225_T225N_T227_14_Opensource_dts.zip` in release `13155`.
- **Unknown:** which region DTS/DTBO is actually active on an SKZ device at runtime (no read-only device inspection was performed).
- **Unknown:** the source of the `-28847731-abT225XXSBEYE4` local-version string in a build tree.
- **Unknown:** whether OSRC release `12636` (T220 `T220XXS9EYA4`) is the exact origin of `4497b689…`; commit message and OSRC listing are consistent, but no archive diff was done.
- **Environment note:** some OSRC queries returned HTTP 520 on repeat; findings rely on successful responses quoted above, not on those error pages.

## 4. Risks / blockers

- Blocking for any exact-source claim: release `13155` archives must be inspected and compared to a candidate commit; not done here by design.
- Blocking for region-correct boot artifacts: no SKZ DTS in the inspected upstream tree; the `_dts.zip` identity is only a name.
- Blocker for reproducibility: unpinned `master` checkout in `.github/workflows/source-preflight.yml`.
- Risk of false confidence: matching `4.19.191` and `MT6765` alone does not establish target compatibility; the patch-line and region differences are material.
- Out-of-scope risks (owned by C2/C4/C3): KSU hook applicability, boot/AVB acceptance, packaging/toolchain consistency.

## 5. Recommended next steps (analysis-only until separately approved)

1. In a later, explicitly authorised analysis step, inspect OSRC release `13155` archive **manifests/file lists** (not firmware) for `gta7lite_defconfig`, `mt8768t_gta7lite*`, and any SKZ DTS; record archive identifiers.
2. Diff those text files against `4497b689…`; if they differ, decide whether to rebase an assessment on the exact BEYE4-derived tree rather than `master`.
3. Pin the assessment ref (commit, not branch) in any future pipeline work; this is C3 ownership, not a change made here.
4. Ask C2 to treat the region DTBO and build-lineage delta as explicit preconditions, and C4 to require non-destructive read-only checks of the device's actual boot/DTBO identity before any boot-image work.
5. If the `_dts.zip` is found to contain an SKZ-specific overlay, re-run the §1.4 table against that overlay before drawing integration conclusions.

## 6. Evidence index

| Source | Locator | Fact supported |
|---|---|---|
| [OSRC exact-build search](https://opensource.samsung.com/uploadSearch?searchValue=T225XXSBEYE4) | release id `13155`, ann. `39127`, 1 row | `T225XXSBEYE4` tied to `SM-T220_14_Opensource.zip` + `SM-T225_T225N_T227_14_Opensource_dts.zip` + `SM-T227U_NA_14_Opensource_dts.zip` |
| [OSRC mobile list](https://opensource.samsung.com/uploadList?classification1=mobile_phone&menuItem=mobile) | `SM-T220` row | `T220XXS9EYA4` → `SM-T220_EUR_14_Opensource.zip` |
| [OSRC T220XXS9EYA query](https://opensource.samsung.com/uploadSearch?searchValue=T220XXS9EYA) | release id `12636` | Separate T220 release cohort (single observed response) |
| [OSRC SM-T225 search](https://opensource.samsung.com/uploadSearch?searchValue=SM-T225) | `T225XXU6DWL9` row | Older T225 source release `SM-T225_EUR_14_Opensource.zip` |
| [Samsung Notify SM-T225](https://doc.samsungmobile.com/SM-T225/018843210521/mlt.html) | `T225XXSBEYE4` entry | Android 14, release `2025-05-29`, patch `2025-05-01` |
| [Samsung Notify SM-T220](https://doc.samsungmobile.com/SM-T220/018911210526/wel.html) | `T220XXS9EYA4` entry | Patch `2025-01-01`, release `2025-02-12` |
| [GitHub commit API](https://api.github.com/repos/gta7lite/android_kernel_samsung_ot8/commits/4497b6897b9d97053bcd99c2551ef2356cb69278) / [HTML](https://github.com/gta7lite/android_kernel_samsung_ot8/commit/4497b6897b9d97053bcd99c2551ef2356cb69278) | `4497b689…` | T220 sync metadata, tree, parent, `HUAQIN_BUILD` diff |
| [GitHub `Makefile` blob](https://github.com/gta7lite/android_kernel_samsung_ot8/blob/4497b6897b9d97053bcd99c2551ef2356cb69278/Makefile#L2-L5) | `:2-5` | `4.19.191` |
| [GitHub `gta7lite_defconfig` blob](https://github.com/gta7lite/android_kernel_samsung_ot8/blob/4497b6897b9d97053bcd99c2551ef2356cb69278/arch/arm64/configs/gta7lite_defconfig) | `:3,22-23,287-291,438-441,1514-1515,1767-1768` | Version header, localversion, MT6765, LTE DTBO, modem config |
| [GitHub LTE DTS blob](https://github.com/gta7lite/android_kernel_samsung_ot8/blob/4497b6897b9d97053bcd99c2551ef2356cb69278/arch/arm64/boot/dts/mediatek/mt8768t_gta7lite_eur_open_00.dts) | `:2,10,14` | `EUR OPEN 00` / `MT8768T` identity |
| [GitHub `setlocalversion`](https://raw.githubusercontent.com/gta7lite/android_kernel_samsung_ot8/4497b6897b9d97053bcd99c2551ef2356cb69278/scripts/setlocalversion) | whole file | No in-tree producer of Samsung-style release suffix |
| [GitHub commit `e5f6bdb…`](https://github.com/gta7lite/android_kernel_samsung_ot8/commit/e5f6bdb50f516c959b8f64e778f092aa3f0cc70a) | 2024-07-26 | Historic T225 `T225XXU6DWL9` import |
| [SamFW](https://samfw.com/firmware/SM-T225/SKZ/T225XXSBEYE4), [SamFrew](https://samfrew.com/download/Galaxy__Tab__A7__Lite__/qjvr/SKZ/T225XXSBEYE4/T225OXMBEYE4) | SKZ pages | Third-party region/build corroboration only |

## 7. Boundary confirmation

No kernel source, config, workflow, previous report, `WAVE.md`, or `SYNTHESIS.md` was modified. No build, patch, package, upload, firmware/archive download, `adb`/`fastboot`/Odin command, flash, unlock, wipe, reboot, or partition operation was performed; no secrets were requested, printed, stored, or used. The only file written by this cell is `.mece/cells/C5-T225/REPORT.md`.
