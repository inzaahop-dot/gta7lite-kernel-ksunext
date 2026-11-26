# C4-VALIDATION — Non-destructive validation and recovery plan

Planning date: 2026-11-26. Scope: staged validation criteria, acceptance criteria, prerequisites, stop conditions, recovery-evidence gaps, and open questions. This cell does **not** audit source provenance, design KernelSU Next integration, or change CI. It is **planning only**: no build, package, flash, unlock, wipe, partition change, or device interaction was performed.

## 0. Inputs, status, and evidence labels

All three required input reports were present and read only:

| Input | Status | Use in this plan |
|---|---|---|
| `.mece/cells/C1-SOURCE/REPORT.md` | Present | Tree/baseline facts; target-identity and source-pinning gaps |
| `.mece/cells/C2-KSUNEXT/REPORT.md` | Present | Compatibility class, integration unknowns, config/symbol gaps |
| `.mece/cells/C3-PIPELINE/REPORT.md` | Present | Build/artifact/packaging baseline and reproducibility gaps |
| `.mece/cells/C4-VALIDATION/TASK.md` | Present | Ownership boundary and deliverable requirements |

Because no input is missing, this plan does not add assumptions to replace absent reports. It still records the **substantive gaps inside** those reports.

Evidence labels used below:

- **VERIFIED (input)** — a fact explicitly recorded in C1, C2, or C3.
- **UNKNOWN** — not established by any input; must not be treated as true.
- **USER INPUT REQUIRED** — only the device owner can supply or confirm it.
- **RECOMMENDATION (future, gated)** — a proposed future criterion or control, not an authorized action.

### 0.1 Verified baseline used for planning (not re-audited here)

| Fact | Input evidence |
|---|---|
| Kernel source is not in this checkout; a preflight workflow fetches `gta7lite/android_kernel_samsung_ot8` at mutable `master`. | C1 `:18-32`; C3 `:33-37` |
| Kernel baseline is `4.19.191`, `arm64`, vendor-heavy `gta7lite_defconfig`; only a preflight workflow exists locally. | C1 `:57-65`; C3 `:11-12,26-40` |
| `CONFIG_KSU` is absent from the inspected defconfigs; no KSU integration has been implemented. | C1 `:74`; C2 `:83,106,171` |
| Target `SM-T225` identity is only partially/conditionally matched; upstream `master` HEAD is a T220 sync; LTE and Wi-Fi variants both exist. | C1 `:84-104,110-112` |
| Upstream KSU Next documents a built-in driver class for 4.19–5.4; GKI 2.0 LKM/prebuilt is version-gated to 5.10+. | C2 `:30-34,101-104` |
| Current design recommendation (C2) is built-in `legacy` + manual hooks; it is a planning choice, not a success claim. | C2 `:5,160-171` |
| No local build command, toolchain pin, packaging script, generated `.config`, boot/recovery image definition, or release artifact exists. | C3 `:12-16,74-89,115-137` |
| Build-time source rewrites are possible under the legacy Kbuild path. | C2 `:67,155-156,216-227` |
| No device evidence, stock recovery assets, rollback artifact, or device access is recorded in any input. | C1/C2/C3 gaps; C3 `:132,137,165` |

---

## 1. Staged validation matrix

Stages are strictly ordered. A later stage is **not authorized** until every earlier gate passes and any separately required approval has been given. Nothing in this matrix is executed by this cell.

| Stage | Purpose | Non-destructive checks / required evidence | Acceptance gate | Stop condition | Authority required |
|---|---|---|---|---|---|
| **0 — Authority and target-evidence gate** | Decide whether any further build or device proposal is even permissible. | Exact model/variant; current firmware/build string; boot image and partition/slot mapping; verified stock recovery assets; known-good rollback path; explicit written approval scope. | All five device-evidence items in §3 are **verified**, not inferred; user confirms scope in writing. | Any missing, ambiguous, or inferred-only item. No device proposal may be made from `gta7lite_defconfig` naming alone. | User/device owner |
| **1 — Static/source checks (read-only)** | Prove that the intended source and config are internally consistent before compiling. | Immutable kernel source commit; KSU source ref/commit; full generated `.config`; required config symbols; exact hook-site function text; SELinux/KDP markers; review of predicted Kbuild rewrites; config-key name consistency; SHA-256 of inputs. | All symbols and signatures required by the chosen integration path are present; no stale guide-only config key is copied; every build-time rewrite is predicted and reviewable; inputs are pinned and hashed. | Missing symbol, signature mismatch, unpinned input, unexplained rewrite, or unresolved guide/source contradiction. | Future implementation phase; read-only only |
| **2 — Compile-time checks (future, isolated)** | Show that the pinned source compiles with the effective toolchain, without packaging or device contact. | Explicit `ARCH=arm64`; chosen defconfig; pinned toolchain with checksum; captured compiler version, full build log, generated `.config`, and post-build source diff; secret scan of logs/outputs. | Clean compile of the expected image target; generated source diff matches Stage 1 prediction; no unexplained dirty files; no secret material in logs/artifacts. | Compile failure, unexplained source mutation, toolchain not reproducible, or secret leakage. | Separate approval for a build phase; no device interaction |
| **3 — Offline artifact inspection** | Verify the produced artifact shape and metadata without installing anything. | Manifest of all pinned SHAs and toolchain checksum; SHA-256 of every output; expected `Image.gz` + DTB/DTBO names; match against Stage 0 target mapping; absence of tokens/keys. | Hashes verify; DTB/DTBO names match the pinned config **and** the verified target mapping; manifest complete; no secrets; artifact shape consistent with Stage 0 evidence. | Hash mismatch, unexpected DT/DTBO, missing manifest entry, or target-shape mismatch. | Future build/artifact phase; internal artifact only |
| **4 — User-supervised device observation (only after 0–3)** | Observe runtime behavior read-only. **No flashing, unlocking, wiping, or partition alteration is authorized by this report.** | Read-only observations of boot behavior, SELinux state, root/manager status, logs, and regression checklist; baseline comparison captured before any change; agreed observation window and boot count. | All acceptance criteria in §2 pass for the full agreed window. | Any hard stop in §2 or §4.2, or any new evidence contradicting Stage 0. | Separate explicit user approval + model-specific verified instructions |
| **5 — Exit and rollback decision point** | Decide whether to accept, stop-and-investigate, or invoke a verified rollback path. | Complete observation record; decision log; rollback artifact and procedure verified **before** they would ever be needed. | Either documented acceptance or documented stop with preserved evidence. | Rollback cannot be invoked because artifacts/compatibility are unverified. | User/device owner; separate approval for any recovery action |

Stage 4 is a **future, user-supervised observation stage** only. This wave does not propose a device test to the user, because the Stage 0 evidence does not exist (§3).

---

## 2. Acceptance criteria for runtime observation (future, gated)

These are criteria to be applied only if a separately approved Stage 4 occurs. They describe **observable outcomes**, not device operations. No command is authorized or executed here; read-only status reads are named only as evidence types.

### 2.1 Boot stability

- **Pass:** the device reaches its normal Android UI within the agreed observation window; no repeated boot loop into recovery/download state; no unexplained watchdog reset or kernel panic; the agreed number of starts all succeed.
- **Evidence to record:** observation window, number of starts, any abnormal state entered, and log evidence of panic/oops if present. The window and count are **user-defined**; a multi-start observation over at least a full day is a RECOMMENDATION, not an instruction.
- **Stop:** any boot loop, boot-to-recovery loop, panic/oops tied to the change, or a start that only succeeds after an unplanned intervention.

### 2.2 SELinux state

- **Pass:** a read-only enforcement-state read (for example the standard `getenforce` status read) reports the expected enforcing state for an unmodified device; no KSU SELinux initialization abort or policy-load failure in logs; no unexplained transition to permissive.
- **Evidence to record:** enforcement state, policy load status, and denial summaries with timestamps.
- **Stop:** permissive state where enforcing was expected, failed policy load, SELinux initialization failure, or denials that break normal system functions.
- **Unknown to resolve first:** C2 `:87,111` records that C1 does not establish `CONFIG_SECURITY`, `CONFIG_SECURITY_SELINUX`, `CONFIG_SECURITYFS`, or vendor SELinux divergence; the expected runtime state must be baselined on the real device before judging a change.

### 2.3 Root functionality

- **Pass:** the manager reports the expected KernelSU Next installation state, and a read-only privilege/identity check confirms the expected root context and access scope; no write tests, module installs, or policy changes are required to establish this.
- **Evidence to record:** manager-reported state, read-only identity/context result, and any root-related crashes or anomalies.
- **Stop:** manager reports no installation, root check fails, privilege scope is unexpected, or obtaining root causes instability.
- **Constraint:** no app mutation, module installation, policy rewrite, or privilege escalation test beyond read-only verification is part of this criterion.

### 2.4 Logging

- **Pass:** read-only kernel/system log capture shows no new `BUG`, `oops`, panic, backtrace, or KSU initialization failure attributable to the change; log volume stays bounded; captured logs contain no secrets or personal credentials.
- **Evidence to record:** log capture window, relevant warnings/errors, and comparison against the pre-change baseline log.
- **Stop:** any new fatal kernel message, repeated KSU/SELinux error, or unbounded error flood.

### 2.5 Regression checks

- **Pass:** a user-defined baseline checklist of normal device functions is unchanged after the observation window. Suggested categories (RECOMMENDATION; not executed here): cellular/modem and SIM services, Wi-Fi, Bluetooth, camera, audio, display/touch, suspend/resume, charging, time/date, and vendor/security indicators that are observable read-only.
- **Evidence to record:** baseline result and post-change result for each checked item, with unknowns marked as not checked.
- **Stop:** any regression in a previously working baseline item, or a function that cannot be checked because required device access is unavailable.
- **Note:** the LTE/Wi-Fi distinction and modem configuration are recorded in C1 `:92,98-100`; therefore "cellular works" cannot be assumed from a defconfig filename and needs device-side baseline evidence.

---

## 3. Prerequisites before any device test is proposed

All items below are **currently UNKNOWN or NOT ESTABLISHED**; none may be filled by inference.

| # | Required evidence | Current status | Why it blocks a device proposal | Source of gap |
|---|---|---|---|---|
| P1 | Exact model and variant (e.g. confirmed regulatory model, region/CSC, LTE vs Wi-Fi, and hardware revision where relevant) | **UNKNOWN** | C1 found no `SM-T225` string; LTE and Wi-Fi variants both exist; current upstream HEAD is a T220 sync. A wrong variant can mean a wrong DT overlay or firmware base. | C1 `:84-104,110-112`; C3 `:129` |
| P2 | Current firmware/build string, bootloader/bootloader-revision, Android version, and patch level actually installed | **USER INPUT REQUIRED** | No report records the installed firmware; Android-version clues are indirect and mixed. | C1 `:72,113`; C3 `:129` |
| P3 | Boot image / partition mapping: active slot(s), boot/recovery/vendor_boot/dtbo mapping, ramdisk policy, and appended-DTB layout as used by the installed firmware | **UNKNOWN** | No local packaging definition exists; C3 explicitly says no model, slot, ramdisk, or recovery assumption is encoded. | C3 `:16,119-122,130-132`; C1 `:64` |
| P4 | Verified stock recovery assets: exact stock boot/recovery/dtbo images (where applicable), source firmware package, checksums, and proof they match P1–P3 | **NOT AVAILABLE / UNVERIFIED** | Without verified stock assets there is no basis for a safe recovery or for comparing artifact shape. No such asset is in the repository or reports. | C3 `:119-122,131`; no input records device assets |
| P5 | Known-good rollback path: verified artifacts + model-specific procedure + confirmed compatibility with the exact current state | **NOT ESTABLISHED** | C4 is forbidden from claiming a recovery path exists without verified artifacts and compatibility. None exists in the inputs. | Task `:19`; C3 `:132,137` |
| P6 | Pinned kernel source commit, KSU source commit, toolchain identity/checksum, and full generated `.config` | **NOT PRESENT** | CI tracks mutable `master`; no build snapshot; toolchain evidence conflicts. No reproducible artifact can be handed to a device phase. | C1 `:32,109,114`; C2 `:179,185`; C3 `:13-16,74-89` |
| P7 | Compile-only success and reviewed Kbuild-generated source diff | **NOT PRESENT** | No integration or build has been performed; runtime testing would be premature. | C2 `:171,188-194`; C3 `:87-89` |
| P8 | Separate explicit user approval and model-specific verified instructions for any eventual flashing/unlocking action | **NOT GIVEN** | Required by task boundary and by C3 handoff rule. | Task `:13,18`; C3 `:153` |

**Gate rule:** a device test may be *proposed* only when P1–P8 are satisfied. Even then, the proposal must be a separate, explicit, model-specific request. This report does not make that proposal.

---

## 4. Stop conditions and rollback decision points

### 4.1 Global stop conditions

Stop all forward progress if any of the following occurs:

- P1–P8 remain unmet or become contradictory.
- Any source, firmware, partition, or recovery claim rests only on filename/inference rather than verified evidence.
- A future static check finds missing required symbols, mismatched hook signatures, unsupported config keys, or unreviewed build-time rewrites.
- A compile, artifact inspection, or observation gate fails.
- New evidence shows the target is not the variant assumed by the selected defconfig/DT overlay.
- Any flashing, unlocking, wiping, or partition change is proposed without separate explicit approval and model-specific verified instructions.

### 4.2 Runtime hard stops (future Stage 4 only)

- Boot loop, panic/oops, or repeated unexpected recovery/download state.
- SELinux not in the expected enforcing state, policy load failure, or KSU SELinux initialization failure.
- Root absent, unexpected privilege scope, or root-related instability.
- New fatal log messages or unbounded error flood.
- Any regression in the pre-change baseline checklist.
- Loss of ability to observe logs or state read-only.

### 4.3 Rollback decision points

1. **Pre-change go/no-go:** confirm P1–P8 before any change is even proposed. If rollback evidence is unverified, the only valid decision is **no change**.
2. **First anomaly:** on any runtime hard stop, freeze further testing and preserve logs/observations. The user decides between "stop and investigate" and "invoke rollback"; rollback is available only if P4 and P5 are verified.
3. **Acceptance review:** after the observation window, accept only if §2 passes; otherwise return to decision point 2.
4. **Recovery execution:** any eventual flashing/unlocking/wipe/recovery action is a **separate action** requiring fresh explicit user approval and model-specific verified instructions. This report neither requests nor describes those operations.

**Current recovery status:** **no verified recovery path is claimed.** P4 and P5 are unmet; therefore rollback availability is UNKNOWN, not "available".

---

## 5. Recovery evidence gaps

| Gap | Evidence needed to close it | Current state | Input basis |
|---|---|---|---|
| Device identity vs artifact identity | Confirm exact model/variant/region and that the DTB/DTBO overlay actually applies to it | Unclosed | C1 `:98-104`; C3 `:129` |
| Stock baseline | Exact installed firmware/build, bootloader state, and read-only pre-change inventory | Unclosed | C1 `:72`; C3 `:129` |
| Partition/slot/ramdisk map | Verified mapping for the exact model and firmware, including boot/recovery/dtbo behavior | Unclosed | C3 `:119-122,130-132` |
| Stock recovery assets | Stock images from a matching firmware package, with checksums and compatibility proof | Unclosed | No input records such assets |
| Rollback procedure | Model-specific verified instructions tied to the exact current state and assets | Unclosed; do not claim availability | Task `:19`; C3 `:132` |
| Reproducible build inputs | Pinned kernel/KSU/toolchain commits, generated `.config`, build manifest and hashes | Unclosed | C1 `:32,109`; C3 `:13-16,74-89` |
| Build-time diff provenance | Reviewable diff of any Kbuild source rewrites | Unclosed | C2 `:67,155-156,216-227` |
| Runtime baseline | Pre-change observations of boot, SELinux, root/manager, logs, and regressions | Unclosed; needs device access | §2 of this report |
| Artifact truth | Downloaded/verified preflight artifact contents and run logs, if they are ever to be relied on | Not inspected by inputs | C3 `:18,162-163` |

These are gaps, not failures of the upstream projects; they define what the user must supply or verify before a device phase can even be proposed.

---

## 6. What cannot be validated without physical-device access

The following remain **UNKNOWN** in this wave and cannot be closed by more repository reading:

1. Exact model/variant, region/CSC, hardware revision, and whether the installed firmware matches the tree's T220-based sync or a true T225 build.
2. Installed firmware/build string, Android version, patch level, bootloader/unlock state, and KNOX/security-fuse indicators.
3. Actual partition layout, active slots, ramdisk inclusion, appended-DTB layout, and boot/recovery/dtbo partition mapping.
4. Existence, integrity, and compatibility of stock boot/recovery/dtbo assets on or for the device.
5. Whether a rollback path actually works; no recovery action or success can be claimed from static evidence.
6. Real boot stability, bootloop behavior, panic/oops occurrence, and recovery-mode entry behavior.
7. Runtime SELinux enforcement state, policy-load behavior, and vendor SELinux/KDP/RKP/DEFEX behavior (C2 `:110,151,183-184`).
8. Root/manager functionality, privilege scope, and absence of privilege-related crashes.
9. Actual kernel/system logs and denial volume.
10. Regressions in modem/LTE, Wi-Fi, Bluetooth, camera, audio, display/touch, suspend/resume, charging, time/date, and vendor features.
11. Applicability of the selected DT overlay and modem/CCCI configuration to the physical unit.
12. OTA behavior, secure-boot verification behavior, and any warranty/security policy consequence of a future change.
13. Any user data preservation outcome; this plan does not assume a backup exists.

---

## 7. Open questions requiring the user

No answer is assumed below. Each item is a prerequisite for a later, separately approved phase.

1. What is the exact regulatory model, variant (LTE vs Wi-Fi), region/CSC, and hardware revision of the device?
2. What firmware build string, Android version, patch level, and bootloader/bootloader-revision are currently installed?
3. Is the device currently stock, or has it already been modified? If modified, what is the exact current state?
4. What is the verified boot/recovery/dtbo partition layout, slot state, and ramdisk policy for that exact firmware?
5. Does the user possess a stock firmware package and stock boot/recovery/dtbo assets matching the exact current state, with checksums? Will they provide the evidence (not secrets or keys) for verification?
6. What is the known-good rollback path, including verified model-specific instructions and artifacts? If none exists, the answer is "none", and the plan stops at Stage 0.
7. Does the user explicitly authorize a future build phase and, later, a separate user-supervised device phase? Who is the named approver for each gate?
8. Should the Wi-Fi variant ever be in scope, or only the LTE configuration selected by the current workflow?
9. What pre-change baseline observations and regression checklist should be used, and what observation window/count defines "stable"?
10. What manager version/expected root scope is acceptable, and what SELinux state is expected on the stock baseline?
11. Are there device functions the user considers critical enough to be automatic stop conditions (for example cellular service or secure-boot indicators)?

---

## 8. Change statement

No project, kernel source, Git history, configuration, workflow, script, build artifact, release, upload, or connected-device state was changed by this cell. No build, package, flash, unlock, relock, wipe, reboot, partition operation, or device command was performed. No `adb`, `fastboot`, Odin, or device-side command was executed, and no stock or modified image was created, downloaded, installed, or verified on a device. All inspection was read-only and limited to the three input reports, the C4 task, and local directory listings. No recovery path is claimed as available. The only file written for this cell is `.mece/cells/C4-VALIDATION/REPORT.md`.
