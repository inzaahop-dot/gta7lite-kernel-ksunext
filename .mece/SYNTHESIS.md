# SYNTHESIS — gta7lite KernelSU Next assessment

Synthesis date: 2026-11-26. Scope: read-only consolidation of the four completed cell reports. This document adds no new source, CI, build, packaging, or device work.

## 0. Synthesis status and decision summary

| Item | Conclusion | Evidence |
|---|---|---|
| Cell reports present | All four `REPORT.md` files exist and are non-empty | `.mece/cells/C1-SOURCE/REPORT.md`, `.mece/cells/C2-KSUNEXT/REPORT.md`, `.mece/cells/C3-PIPELINE/REPORT.md`, `.mece/cells/C4-VALIDATION/REPORT.md` |
| Analysis wave | Complete for its stated analysis-only purpose | Definition of done at `.mece/WAVE.md:39-47` |
| KSU Next integration readiness | **NO-GO** for implementation, build, packaging, release, or device work | See §4 and §7 |
| Permitted next activity | Evidence collection and read-only static verification, only after separate explicit approval | C4 `:43-54,97-112`; C3 `:141-153` |
| Device state | No device evidence, stock assets, or verified rollback path exists in the inputs | C4 `:97-112,138-163` |
| Recovery claim | **No verified recovery path is claimed or available on the evidence provided** | C4 `:145` |

The wave produced a usable assessment plan, not a validated integration. The fact that KernelSU Next documents a compatible kernel class for `4.19.191` arm64 does not establish that this vendor tree, this defconfig, this target device, or any future artifact will work.

---

## 1. Overall readiness assessment

| Dimension | Readiness | Basis |
|---|---|---|
| Source baseline | **Partial** | Kernel source is not in the checkout; CI fetches mutable `master`, and only a runtime-recorded HEAD is available (C1 `:15-18,24-32,109`). |
| Target identity | **Not ready** | `SM-T225` is not proven in the selected defconfig/DT files; LTE and Wi-Fi variants exist and current upstream HEAD is a T220 sync (C1 `:84-104,110-112`). |
| KSU compatibility class | **Plausible in class, unproven on target** | Upstream supports `4.4-6.18` and requires built-in driver for `4.19-5.4`; GKI LKM/prebuilt is `5.10+` only (C2 `:30-36,101-104`). |
| Integration design | **Planning default only** | Built-in `legacy` + manual hooks is the recommended future candidate, explicitly not a success claim (C2 `:5,160-171`). |
| Build pipeline | **Not ready** | Only a `workflow_dispatch` preflight exists; no build command, toolchain pin, generated `.config`, packaging definition, release, or reproducible artifact baseline exists (C3 `:11-18,40-49,74-89,115-137`). |
| Validation and rollback | **Not ready** | P1-P8 prerequisites are unmet; no device proposal is authorized and no rollback path is verified (C4 `:97-112,138-163`). |
| Authorization | **Not ready for implementation** | Current wave is analysis-only; each future build/device stage requires separate explicit approval (C3 `:141-153`; C4 `:43-54,110-112`). |

**Overall:** the assessment wave is complete; the integration is not ready to proceed. Any forward movement must begin with user-supplied device/target evidence and a separately approved, read-only static verification stage.

---

## 2. Findings from all four cells

### 2.1 C1-SOURCE — source and target baseline

- The local project contains only `README.md`, `.github/workflows/source-preflight.yml`, and `.mece/.../*.md` planning/task files; the kernel source is not present (C1 `:15-18`).
- The preflight workflow is dispatch-only, uses `contents: read`, checks out `gta7lite/android_kernel_samsung_ot8` at mutable `master` into `kernel-source`, verifies `arch/arm64/configs/gta7lite_defconfig`, and uploads the defconfig plus source commit (C1 `:24-32,43-51`).
- Observed upstream `master` HEAD was `4497b6897b9d97053bcd99c2551ef2356cb69278`, with message `kernel: sync T220XXS9EYA4`; the workflow does not pin that commit (C1 `:40-49,109`).
- The selected baseline is Linux `4.19.191`, `arm64`, `CONFIG_ARCH_MEDIATEK=y`, `CONFIG_MACH_MT6765=y`, `CONFIG_MTK_PLATFORM="mt6765"`, project `ot8`; the LTE defconfig selects the `mt8768t_gta7lite_eur_open_00` DT overlay and enables modem/CCCI (C1 `:57-65,88-93`).
- `CONFIG_KSU` is absent from the inspected LTE and Wi-Fi defconfigs; `CONFIG_KPROBES`, `CONFIG_UPROBES`, and `CONFIG_KRETPROBES` are enabled (C1 `:74-75`).
- Target identity remains conditional: no `SM-T225` string was found; both LTE and Wi-Fi configs/DTs exist; the upstream head sync names T220 (C1 `:84-104,110-112`).
- Toolchain evidence conflicts: the generated defconfig records GCC 14.1.1 while MTK build configs specify LLVM/Clang (C1 `:65,73,114`).
- Provenance and licensing limits remain: no Samsung release linkage, vendor-generated DTS files lack individual SPDX headers, and GitHub classifies the source license as `NOASSERTION` (C1 `:77-82,116-117`).

### 2.2 C2-KSUNEXT — upstream compatibility and integration points

- Upstream subject is `KernelSU-Next/KernelSU-Next`, default branch `dev`; observed `dev`, `legacy`, and `samsung-lateload` HEADs and release tag `v3.4.2` are recorded (C2 `:15-22`).
- Documentation supports kernels `4.4-6.18`; `4.19-5.4` requires a built-in driver; GKI 2.0 prebuilt/LKM/KMI is documented for `5.10+` only (C2 `:30-36`).
- Confirmed incompatible for this baseline: the GKI 2.0 LKM/prebuilt path and the guide's `CONFIG_KSU_KPROBE_HOOKS` key versus current source symbols (C2 `:104,109,117`).
- Design recommendation is built-in `legacy` with `CONFIG_KSU=y` and manual hooks; syscall-table hooks are a secondary candidate; kprobe mode is validation-only; `samsung-lateload` is a research lead only (C2 `:43-49,160-171`).
- Driver wiring would use `drivers/kernelsu`, `drivers/Makefile`, `drivers/Kconfig`, and `gta7lite_defconfig`; manual hook sites are in `fs/exec.c`, `fs/open.c`, `fs/read_write.c`, `fs/stat.c`, and `kernel/reboot.c` (C2 `:123-140,203-215`).
- Significant unknowns remain: `EXT4_FS`, `KPROBE_EVENTS`, `HAVE_SYSCALL_TRACEPOINTS`, SELinux/LSM configurations, Samsung KDP/RKP/DEFEX markers, UAPI header/preinit behavior, and exact hook signatures are not established by C1 (C2 `:87,105-115,180-186`).
- Legacy Kbuild can rewrite kernel/SELinux source during build, including backports and vendor detection; this makes review of generated diffs mandatory before compile acceptance (C2 `:67,155-156,216-227`).
- Upstream guidance drift is real: website/source config names disagree and installation-guide refs are inconsistent (C2 `:51-59,181`).

### 2.3 C3-PIPELINE — build and artifact pipeline

- Only one workflow exists: `.github/workflows/source-preflight.yml`; it is a manual inspection workflow, not a build or release pipeline (C3 `:11,26-40,46-49`).
- No local shell scripts, Makefile, Dockerfile, dependency manifest, toolchain pin, submodules, packaging definition, or build documentation exists (C3 `:12,40,119-123`).
- Inputs are not pinned: moving action tags, `ubuntu-latest`, mutable kernel `master`, shallow clone, and no toolchain pin (C3 `:13-16,74-89`).
- Per-run provenance is limited to a defconfig copy and kernel source HEAD; no project SHA, action SHAs, runner image, compiler version, generated `.config`, command line, or output hash is recorded (C3 `:14,43-45,74-89`).
- Upstream build configs disagree on compiler/prebuilt paths and reference `prebuilts/...` directories absent at the pinned root; default kernel outputs are `Image.gz` plus DTB/DTBO, but Android boot/recovery packaging is undefined locally (C3 `:51-70,115-137`).
- Remote observations: two workflow runs and one preflight artifact with an API-reported digest; artifact contents and run logs were not inspected (C3 `:18,44-49,162-163`).
- A staged future pipeline was proposed with pinning, compile-only checks, manifest/hash verification, offline inspection, a manual release gate, and a separate device phase; none is implemented or authorized (C3 `:141-155`).

### 2.4 C4-VALIDATION — non-destructive validation and recovery planning

- C4 consumed all three prior reports and defined evidence labels so that inference cannot be substituted for verification (C4 `:5-16,18-24`).
- The staged matrix is ordered: authority/target evidence, read-only static checks, compile-only checks, offline artifact inspection, user-supervised observation, and exit/rollback decision (C4 `:41-54`).
- Runtime acceptance criteria are observable outcomes only: boot stability, SELinux state, manager/root state, logs, and user-defined regression checks (C4 `:58-94`).
- Prerequisites P1-P8 are all unmet: exact model/variant, firmware state, partition/slot/ramdisk mapping, stock assets, rollback path, pinned inputs, compile success/diff review, and explicit approval (C4 `:97-112`).
- Global and runtime stop conditions are defined; any flashing, unlocking, wiping, or partition action requires separate explicit approval and model-specific verified instructions (C4 `:116-143`).
- Recovery status is explicitly negative: **no verified recovery path is claimed**; rollback availability is UNKNOWN (C4 `:138-163`).
- The report asks the user eleven unresolved questions and does not propose a device test (C4 `:54,187-201`).

---

## 3. Cross-cell dependencies and contradictions

### 3.1 Dependencies

| Dependency | Consequence |
|---|---|
| C1 establishes the kernel/config baseline used by C2 and C3 | Because C1 found no committed source snapshot, C2 hook-signature conclusions and C3 build inputs remain conditional. |
| C2 identifies config symbols, hook sites, SELinux rewrites, and KSU ref requirements | These become Stage 1 static checks in C4 and mandatory inputs to any future C3 compile stage. |
| C3 establishes that no build/packaging baseline exists | C4 cannot accept an artifact until pinning, compile-only success, manifest hashes, and target-shape verification exist. |
| C4 establishes that device evidence and rollback are absent | Even a successful future build cannot justify a device proposal; Stage 0 is a hard gate. |
| All cells depend on user authority and evidence | Target identity, firmware, partition/recovery facts, and approvals are outside repository inspection. |

### 3.2 Contradictions, tensions, and how they are resolved here

1. **Worktree state discrepancy.** C1 recorded branch `mece/kernel-source-audit-plan` and HEAD `c7f281cf36e3a07757e3158b99ce328d820e01c1` (C1 `:10-12`); the parent observed the current worktree on `report/c4-validation-audit` at `cc5609bcc1624f96e2fcbb78238903613a9dd83f`, with `C1-SOURCE/REPORT.md` untracked. This is treated as an audit-time/current-state difference, not proof that kernel facts changed. It remains unresolved by this wave; no Git state was altered here.
2. **Wave status text versus execution state.** `.mece/WAVE.md:3-5` still says `PLANNED` and that OpenCode has not been started, while all four reports exist and the parent task explicitly requests synthesis. The status header is treated as stale planning text; this document performs only the explicitly requested synthesis deliverable.
3. **Target label versus evidence.** The plan names `SM-T225` LTE, but C1 found no in-tree `SM-T225` string and recorded a T220-based sync plus LTE/Wi-Fi variants (C1 `:84-104`). The target is therefore unverified; filename-based inference is rejected.
4. **Toolchain evidence conflict.** Defconfig header says GCC 14.1.1; upstream build configs say LLVM/Clang with absent prebuilt paths (C1 `:73`; C3 `:51-70`). No compiler is selected by this synthesis.
5. **Kprobe guidance conflict.** The non-GKI guide recommends kprobes when reliable, while current legacy Kconfig warns against use below `5.10` and uses different config names (C2 `:41-48,51-59,108`). Baseline `CONFIG_KPROBES=y` does not resolve this; kprobe mode remains validation-only.
6. **Recommendation versus authorization.** C2's manual-hook recommendation is a design default, not authority to patch or build; C3 and C4 impose separate gates. There is no contradiction: implementation remains blocked.
7. **Historical workflow trigger.** C3 observed an old failed run on push at an earlier workflow revision, while the current definition is dispatch-only (C3 `:46-49,97`). The current definition, not the historical revision, governs present automation.
8. **Artifact digest versus artifact truth.** C3 recorded an API-reported artifact digest but did not download or verify contents (C3 `:18,162-163`). The artifact is not used as evidence for target or build correctness.

No report contradicts another on the central conclusion: facts are incomplete and forward work is gated. The tensions above are unresolved evidence conflicts, not reasons to proceed.

---

## 4. Prioritized blockers

| Priority | ID | Blocker | Evidence | Gate until resolved |
|---|---|---|---|---|
| P0 | B1 | Exact model/variant, region, firmware, and hardware state are unverified | C1 `:84-104,110-112`; C4 `:103-104,191-193` | No device proposal or packaging decision |
| P0 | B2 | No verified stock assets, partition/slot/ramdisk mapping, or rollback path exists | C3 `:119-132`; C4 `:105-107,138-163` | No flashing, recovery claim, or device phase |
| P0 | B3 | No separate explicit approval for a build phase or device phase | `.mece/WAVE.md:15-23`; C3 `:141-153`; C4 `:110-112` | No implementation, build, release, or device work |
| P1 | B4 | Kernel source is mutable/unpinned and no generated `.config` or full input snapshot exists | C1 `:32,109`; C3 `:13-16,74-89`; C4 `:108` | No reliable hook-site, SELinux, or artifact assessment |
| P1 | B5 | Toolchain is unresolved and upstream configs reference absent prebuilt paths | C1 `:65,73,114`; C3 `:51-70,83` | No reproducible compile-only stage |
| P1 | B6 | Required KSU config symbols, hook signatures, SELinux/LSM settings, Samsung markers, and header/preinit behavior are unknown | C2 `:87,105-115,180-186` | No KSU patch design can be accepted as complete |
| P1 | B7 | No local build, packaging, manifest, or artifact-shape baseline exists | C3 `:12-16,74-89,115-137` | No artifact can be claimed device-appropriate |
| P2 | B8 | KSU upstream guidance drift and legacy build-time source rewrites require explicit review | C2 `:51-59,67,155-156,216-227` | No compile acceptance without reviewed generated diffs |
| P2 | B9 | Samsung/MediaTek hardening and vendor SELinux divergence may invalidate hook strategy | C1 `:69-71`; C2 `:107,110,183-184` | No selection of syscall-table or kprobe mode without evidence |

---

## 5. Staged non-destructive roadmap and approval gates

All stages are proposals for future, separately approved phases. Nothing below has been performed by this synthesis.

| Stage | Non-destructive work | Gate to next stage | Stop condition |
|---|---|---|---|
| **A — User evidence and authority** | User supplies/confirms exact model and variant, firmware/build string, Android version/patch level, partition/slot/ramdisk/recovery mapping, stock asset evidence, and rollback evidence; user names approver and defines scope. | B1-B3 resolved in writing; target decision explicitly LTE `gta7lite` or different variant. | Any missing, ambiguous, or inferred-only item. No device proposal may be made from defconfig names. |
| **B — Read-only static verification** | Pin and hash kernel source commit, KSU source ref/commit, action/toolchain identities; capture full generated `.config`; verify `EXT4_FS`, kprobe/tracepoint symbols, SELinux/LSM configs, Samsung markers, UAPI header behavior, exact hook-site text, and config-key names; predict all legacy Kbuild rewrites. | Named reviewer approves a pinned manifest and static findings; separate approval granted for a compile-only phase. | Missing symbol, signature mismatch, stale guide-only key, unexplained rewrite, or unresolved source ref. |
| **C — Compile-only build (future, isolated)** | With pinned toolchain and checksum, build the selected defconfig; capture compiler version, build log, `.config`, generated source diff, input/output hashes, and secret scan. No packaging, upload, or device interaction. | Clean compile; generated diff matches Stage B prediction; no unexplained dirty files; no secret material in outputs. | Compile failure, unexplained source mutation, non-reproducible toolchain, or secret leakage. |
| **D — Offline artifact inspection** | Verify manifest hashes, `Image.gz` + DTB/DTBO names, DTB/DTBO applicability to Stage A target evidence, and absence of tokens/keys. Artifact remains internal. | Artifact shape matches verified target mapping and manifest is complete. | Hash mismatch, unexpected DT/DTBO, missing manifest entry, or target-shape mismatch. |
| **E — Device phase (future, user-supervised, read-only observation only)** | Only after A-D and fresh explicit approval: baseline read-only observations, then observe boot, SELinux state, manager/root status, logs, and user-defined regression checklist. No flashing, unlocking, relocking, wiping, rebooting, partition changes, module installs, or device-side root commands are authorized here. | All C4 §2 criteria pass for the user-defined observation window. | Any global/runtime hard stop in C4 `:118-136`; any new evidence contradicting Stage A. |
| **F — Exit, rollback, or optional distribution decision** | Document accept/stop/investigate; rollback only if stock assets and procedure were verified in Stage A and fresh explicit approval is given. Optional release/distribution requires its own protected-environment gate. | Documented acceptance or documented stop with preserved evidence. | Rollback unverified; distribution or recovery action proposed without separate approval. |

**Approval gates are cumulative.** Skipping any gate is a stop condition. This wave authorizes none of stages B-F.

---

## 6. Verified facts versus unknowns and required user evidence

### 6.1 Verified facts

| Fact | Evidence |
|---|---|
| Kernel source is absent from the checkout and fetched at workflow runtime from mutable `master` | C1 `:18,24-32`; C3 `:33-37` |
| Baseline observed at upstream `4497b689` is Linux `4.19.191`, arm64, MediaTek MT6765/`ot8`, vendor-heavy `gta7lite_defconfig` | C1 `:40-65` |
| `CONFIG_KSU` is absent from inspected defconfigs; `KPROBES`/`UPROBES`/`KRETPROBES` are enabled | C1 `:74-75` |
| Upstream KSU Next documents `4.4-6.18` support and a built-in-driver class for `4.19-5.4`; GKI LKM/prebuilt is `5.10+` only | C2 `:30-36,101-104` |
| Current design candidate is built-in `legacy` + manual hooks, explicitly unvalidated on this target | C2 `:5,160-171` |
| Only a dispatch-only preflight workflow exists; no local build/packaging baseline or toolchain pin exists | C3 `:11-18,40-49,74-89,115-123` |
| Target identity, firmware state, stock assets, rollback, and device observations are not established | C1 `:98-104`; C3 `:129-132`; C4 `:97-112,167-183` |
| No source/config/workflow/report/Git-history/build/device change was made by the four cells | C1 `:141-143`; C2 `:280-282`; C3 `:191-193`; C4 `:205-207` |

### 6.2 Unknowns that must not be treated as facts

- Whether the physical unit is actually `SM-T225`, and whether it is LTE or Wi-Fi, its region/CSC, hardware revision, or firmware base (C1 `:84-104`; C4 `:103-104`).
- Installed firmware build string, Android version, patch level, bootloader state, KNOX/security indicators, and current modified/stock state (C1 `:72,113`; C4 `:104,171-172,191-193`).
- Actual partition layout, active slots, boot/recovery/vendor_boot/dtbo mapping, ramdisk policy, appended-DTB layout, and secure-boot behavior (C3 `:119-132`; C4 `:105-107,173`).
- Existence, integrity, and compatibility of stock boot/recovery/dtbo assets and any rollback path (C4 `:106-107,145,154-160`).
- Effective compiler and flags; exact generated `.config`; `EXT4_FS`, `KPROBE_EVENTS`, `HAVE_SYSCALL_TRACEPOINTS`, SELinux/LSM configs, Samsung KDP/RKP/DEFEX markers, `CONFIG_HEADERS_INSTALL`, and hook signatures (C1 `:73,114`; C2 `:87,105-115`).
- Whether legacy Kbuild rewrites apply cleanly to the pinned vendor tree and whether Samsung write protections would affect syscall-table hooks (C2 `:67,107,155-156,183-184`).
- Runtime boot stability, SELinux enforcement, manager/root functionality, logs, and regressions (C4 `:58-94,176-181`).
- Contents of the existing preflight artifact and historical run logs; neither was inspected or relied on (C3 `:18,162-163`).

### 6.3 Required user evidence

The user/device owner must provide or verify, without sharing secrets or keys:

1. Exact regulatory model, variant (LTE vs Wi-Fi), region/CSC, and relevant hardware revision.
2. Installed firmware/build string, Android version, patch level, bootloader/bootloader-revision, and whether the device is stock or modified.
3. Verified boot/recovery/dtbo partition layout, slot state, ramdisk policy, and appended-DTB behavior for that exact firmware.
4. Evidence of matching stock firmware/assets with checksums, and proof they match items 1-3.
5. A known-good, model-specific rollback artifact and procedure, or an explicit answer that none exists.
6. Written scope and named approver for any future static-verification phase, compile-only phase, and later user-supervised device phase.
7. Decision whether Wi-Fi variant support is ever in scope.
8. User-defined regression checklist, observation window/start count, expected stock SELinux state, expected manager/root scope, and any automatic stop conditions.

No password, PAT, private key, signing key, or other secret is requested or needed for this roadmap.

---

## 7. Explicit decision whether KSU Next integration is ready

**Decision: NO-GO. KernelSU Next integration is not ready to proceed beyond assessment and planning.**

Rationale: the integration rests on an unpinned source ref, an unverified device target, unresolved KSU config/symbol/SELinux questions, no toolchain or packaging baseline, no compile evidence, and no verified rollback path or authorization. The upstream kernel-version class is compatible in principle, and the proposed built-in `legacy` + manual-hook design is the most defensible candidate, but neither fact constitutes target readiness.

The only permitted continuation is a separately approved, read-only evidence and static-verification stage (Stage A/B in §5). Any build, patch, artifact publication, or device interaction requires fresh explicit approval at its own gate.

---

## 8. Exact change statement

This synthesis step created exactly one file: `/root/gta7lite-kernel-ksunext/.mece/SYNTHESIS.md`.

No kernel source, defconfig, config, workflow, script, cell report, task file, tracked project file, Git history, branch, commit, tag, build artifact, release, upload, package, or connected-device state was created, modified, staged, committed, or deleted by this synthesis step. The four cell reports and `.mece/WAVE.md` were read only; no report was edited. No build, packaging, flash, unlock, relock, wipe, reboot, partition operation, release, deployment, `adb`, `fastboot`, Odin, device-side command, or root command was performed. No secret value was requested, accessed, printed, stored, or used.

Note on pre-existing state observed read-only at synthesis time: the current worktree was on `report/c4-validation-audit` at `cc5609bcc1624f96e2fcbb78238903613a9dd83f`, and `.mece/cells/C1-SOURCE/REPORT.md` was untracked. This synthesis did not stage, commit, move, or alter that file or any other Git state.
