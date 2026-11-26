# C3-PIPELINE — Build and artifact pipeline audit

Audit date: 2026-11-26. Scope: existing automation, toolchain declarations, build inputs, artifact handling, and packaging assumptions for `inzaahop-dot/gta7lite-kernel-ksunext`. This cell does **not** own kernel-source/KSU integration design or physical-device validation. Sections 1–4 describe **existing behavior** observed during this read-only audit; section 5 is explicitly a **recommendation** for a future, separately approved build.

No workflow, script, build, release, upload, or device action was started or modified during this audit. No secret values were read or printed.

## 0. Executive summary of existing behavior

| Topic | Finding | Evidence |
|---|---|---|
| Local automation | One workflow only: `.github/workflows/source-preflight.yml`. It is a manual **preflight/inspection** workflow, not a build or release pipeline. | `.github/workflows/source-preflight.yml:1-51`; GitHub Actions workflows API (`total_count: 1`) |
| Local scripts/toolchain/build/packaging | No local shell scripts, Makefile, Dockerfile, dependency manifest, submodules, or packaging definitions are present. `README.md` has no build instructions. | local file listing; `git ls-files`; `README.md:1-2`; C1 `REPORT.md:15-18` |
| Reproducibility | Not reproducible from pinned inputs: moving action tags, moving runner label, mutable `master` source ref, shallow clone, no toolchain pin, no committed source snapshot. | `.github/workflows/source-preflight.yml:11,14,17,20-22,48` |
| Provenance | The only per-run provenance written into an artifact is the fetched kernel source `HEAD` and a copy of the defconfig. Project commit, action SHAs, toolchain version, generated `.config`, and output hashes are absent. | `.github/workflows/source-preflight.yml:43-45` |
| Permissions/secrets | Top-level `contents: read`; dispatch-only; no `secrets.*` references and no `env:` block. No write scopes. No secret values were inspected. | `.github/workflows/source-preflight.yml:3-7`; grep over `.github` for `secrets.` and `env:` (no matches) |
| Packaging | No local packaging script exists, so no local device-model, partition-layout, slot, ramdisk, or recovery assumption is encoded. Upstream build configs imply `Image.gz` + DTB/DTBO outputs but do not define Android boot/recovery image packaging. | `.github/workflows/source-preflight.yml:24-29,43-51`; upstream `Makefile:2-5`, `arch/arm64/Makefile:127-132`, `arch/arm64/configs/gta7lite_defconfig:438-441`, `arch/arm64/boot/dts/mediatek/Makefile:1-15` @ `4497b689` |

Remote metadata observed read-only: two workflow runs exist (#1 failed at the initial empty-workflow commit `804d368`, #2 succeeded via `workflow_dispatch` on `main`/`0ba1c04b`), and one artifact named `gta7lite-source-preflight` exists with API-reported digest `sha256:4e9f24b60fa70e7ff78cf5d3907aeb14540fc9ddf40a61e1e1be7855f211dd7c`. The releases and tags APIs returned empty lists. Artifact contents were not downloaded, and run logs were not inspected. Sources: GitHub workflows, runs, artifacts, releases, and tags API endpoints for this repository.

---

## 1. Workflow and script inventory (existing behavior)

### 1.1 Local project inventory

| Path | Lines | Role |
|---|---|---|
| `.github/workflows/source-preflight.yml` | 1 | Workflow name `Kernel source preflight`. |
| `.github/workflows/source-preflight.yml` | 3-4 | Trigger: `workflow_dispatch` only. Quoted `'on':` avoids YAML 1.1 boolean-key parsing. |
| `.github/workflows/source-preflight.yml` | 6-7 | Top-level `permissions: contents: read`. |
| `.github/workflows/source-preflight.yml` | 10-11 | Single job `inspect-source` on `ubuntu-latest`. |
| `.github/workflows/source-preflight.yml` | 13-14 | Checkout of this repository with `actions/checkout@v4`. |
| `.github/workflows/source-preflight.yml` | 16-22 | Checkout of `gta7lite/android_kernel_samsung_ot8` at `ref: master` into `kernel-source`, `fetch-depth: 1`. |
| `.github/workflows/source-preflight.yml` | 24-29 | Bash step asserting `arch/arm64/configs/gta7lite_defconfig` exists. |
| `.github/workflows/source-preflight.yml` | 31-41 | Prints kernel version fields, source `HEAD`, selected defconfig keys, and `CONFIG_KSU` grep results. |
| `.github/workflows/source-preflight.yml` | 43-45 | Creates `preflight-output/`, copies the defconfig, and writes `source-commit.txt`. |
| `.github/workflows/source-preflight.yml` | 47-51 | Uploads `preflight-output/` via `actions/upload-artifact@v4` as `gta7lite-source-preflight`. |
| `README.md` | 1-2 | Project title and `For me`; no build, toolchain, or artifact documentation. |

Inventory method: local `find` over all non-`.git` files and `git ls-files -s` returned only the workflow, `README.md`, and `.mece/*.md` planning/task files. There is **no** local `*.sh`, `Makefile`, `Dockerfile`, `.gitmodules`, `.github/dependabot.yml`, toolchain lockfile, AnyKernel/repack script, or boot-image packaging script in scope. The tracked workflow blob is `c411d99acdc9bfad55f7ed51450e81d7a4d0be48`, matching the workflow API's `main` blob. Workflow history: `804d368` (empty file), `d079250` (workflow body added), `0ba1c04` (`on:` quoted). All Git inspection was read-only.

### 1.2 Remote automation state (existing behavior)

| Observation | Evidence |
|---|---|
| Exactly one active workflow named `Kernel source preflight`, path `.github/workflows/source-preflight.yml`. | GitHub Actions workflows API (`total_count: 1`) |
| Two runs: #1 failed on push at empty-workflow commit `804d368`; #2 succeeded on `0ba1c04` via `workflow_dispatch`. Failure cause of #1 was not inspected. | GitHub Actions runs API |
| One artifact from run #2 (`2026-11-26T09:57:37Z`), `expires_at` 2027-02-24; no `retention-days` is set in the workflow (`:47-51`), so platform default retention applies. | GitHub Actions artifacts API; `.github/workflows/source-preflight.yml:47-51` |
| No releases and no tags. | GitHub releases/tags APIs |

### 1.3 Upstream toolchain/build declarations relevant to this workflow

These files are **not** local project scripts and were read only at pinned commit `4497b6897b9d97053bcd99c2551ef2356cb69278`. They are fetched upstream by the workflow at runtime (`.github/workflows/source-preflight.yml:16-22`); none is invoked for compilation by the local workflow.

| Upstream path @ `4497b689` | Lines | Declaration / implication |
|---|---|---|
| `build.config.mtk.aarch64.tiramisu` | 1-8 | `ARCH=arm64`, `CROSS_COMPILE=aarch64-linux-gnu-`, `LLVM=1`, `LLVM_IAS=1`, `LD_LIBRARY_PATH`/`CLANG_PREBUILT_BIN` pointing to `prebuilts/clang/host/linux-x86/clang-r433403b/...`, `KERNEL_DIR=kernel-4.19`. |
| `build.config.mtk.aarch64.tiramisu` | 10-18 | LLVM tools: `CC=clang`, `LD=ld.lld`, `AR=llvm-ar`, `NM=llvm-nm`, `OBJCOPY=llvm-objcopy`, etc. |
| `build.config.mtk.aarch64.gcc` | 1-6 | GCC variant: `ARCH=arm64`, `CC=gcc`, `KERNEL_DIR=kernel-4.19`, prebuilt path `prebuilts/gcc/linux-x86/aarch64/aarch64-linux-gnu-6.3.1/bin`. |
| `build.config.mtk.aarch64` | 1-18 | Another LLVM config using `clang-r383902` paths. Three toolchain configs disagree on compiler/prebuilt version. |
| `Makefile` | 2-5 | Kernel version `4.19.191`, empty `EXTRAVERSION`. |
| `arch/arm64/Makefile` | 127-132 | Default build target is `Image.gz` plus `dtbs`; `KBUILD_IMAGE := $(boot)/Image.gz`, `KBUILD_DTBS := dtbs`. |
| `arch/arm64/configs/gta7lite_defconfig` | 7-11 | Generated-file header records `gcc (GCC) 14.1.1 20240720`, `CONFIG_CC_IS_GCC=y`, `CONFIG_CLANG_VERSION=0`. |
| `arch/arm64/configs/gta7lite_defconfig` | 22-23 | `CONFIG_LOCALVERSION=""`, `CONFIG_LOCALVERSION_AUTO=y` — version string can vary with Git state. |
| `arch/arm64/configs/gta7lite_defconfig` | 438-441 | `CONFIG_BUILD_ARM64_APPENDED_DTB_IMAGE=y`, `CONFIG_BUILD_ARM64_DTB_OVERLAY_IMAGE=y`, appended DTB `mediatek/mt6765`, overlay `mediatek/mt8768t_gta7lite_eur_open_00`. |
| `arch/arm64/boot/dts/mediatek/Makefile` | 1-15 | Expands those config names into `.dtb`/`.dtbo` outputs. |
| `arch/arm64/boot/install.sh` | 37-60 | Host-side `make install` helper: honors `~/bin/${INSTALLKERNEL}` / `/sbin/${INSTALLKERNEL}`, writes `vmlinuz-$1`/`System.map-$1` under `$4`, rotating `.old` copies. It does not create Android boot/recovery images and is not referenced by the local workflow. |
| Root contents @ `4497b689` | JSON `name` list (67 entries) | No root-level `prebuilts/` directory exists, although build configs reference `prebuilts/clang/...` and `prebuilts/gcc/...`. `scripts/` and `tools/` contain kernel build helpers, not Android packaging definitions. |

Upstream raw-file URLs used: `https://raw.githubusercontent.com/gta7lite/android_kernel_samsung_ot8/4497b6897b9d97053bcd99c2551ef2356cb69278/<path>`; root listing: GitHub contents API with `ref=4497b689`.

---

## 2. Reproducibility and provenance findings (existing behavior)

| ID | Finding | Evidence | Impact |
|---|---|---|---|
| R1 | The project commit is not an input to the workflow. `workflow_dispatch` has no `ref`/`inputs` declaration, and no commit is pinned in the YAML. Run #2 recorded `0ba1c04` only after execution. | `.github/workflows/source-preflight.yml:3-4`; GitHub runs API | The same workflow definition can run against different project commits unless the operator records the branch manually. |
| R2 | Kernel source is checked out from moving branch `master`, not a commit or tag. | `.github/workflows/source-preflight.yml:16-22` | Inputs can change between runs; no build can be reproduced from the YAML alone. |
| R3 | Shallow clone (`fetch-depth: 1`) discards history and tags. | `.github/workflows/source-preflight.yml:22` | `CONFIG_LOCALVERSION_AUTO=y` (`gta7lite_defconfig:22-23`) can produce version strings that depend on clone depth/state; tag-based provenance is unavailable. |
| R4 | All third-party actions use moving major tags (`@v4`), not immutable commit SHAs. | `.github/workflows/source-preflight.yml:14,17,48` | Action code can change without a repository commit; supply-chain and reproducibility risk. |
| R5 | Runner is `ubuntu-latest`, a moving image label; no container digest or OS version is declared. | `.github/workflows/source-preflight.yml:11` | Host tools (`git`, `grep`, `cp`, shell) and libraries can change between runs. |
| R6 | No toolchain setup or pin exists in the project workflow. Upstream build configs point to three different compiler/prebuilt paths that are absent from the pinned source tree; the selected defconfig header records GCC 14.1.1. | `.github/workflows/source-preflight.yml:12-45`; upstream `build.config.mtk.aarch64.tiramisu:1-18`, `build.config.mtk.aarch64.gcc:1-6`, `build.config.mtk.aarch64:1-18`, `gta7lite_defconfig:7-11`; root contents API | Effective compiler is unresolved. A future build must choose and pin a toolchain explicitly. |
| R7 | No build command (`make`/`build.sh`), no generated `.config`, no compiler/version capture, no hashes, and no dependency cache key appear in the workflow. | `.github/workflows/source-preflight.yml:24-45` | The workflow cannot produce a reproducible kernel artifact today; it only inspects inputs. |
| R8 | Artifact provenance is limited to the defconfig copy and source `HEAD`. No project SHA, action SHAs, runner image, compile command, toolchain version, `.config`, or output digest is stored. | `.github/workflows/source-preflight.yml:43-45`; GitHub artifacts API | A third party cannot reconstruct or verify the exact inputs from the artifact alone. |
| R9 | Credentials persistence is not disabled for either checkout step; action defaults apply. | `.github/workflows/source-preflight.yml:13-17` | With read-only `GITHUB_TOKEN` and public repositories the current risk is low, but it is an implicit dependency/credential surface worth removing. |
| R10 | Observed run history contains no build, packaging, release, or deployment run; releases and tags are empty. | GitHub runs/releases/tags APIs | There is no existing reproducible build baseline to inherit. |

Conclusion: **builds are not reproducible from pinned inputs** because no build is defined and the inputs that are fetched are mutable. The existing workflow provides a partial, runtime-generated provenance record for the kernel source only.

---

## 3. Permissions and secret usage

### 3.1 Existing behavior

- `.github/workflows/source-preflight.yml:3-4` allows only manual dispatch. No `push`, `pull_request`, `schedule`, `release`, or `workflow_call` trigger is declared, so untrusted PR code cannot trigger this workflow directly. (Historical run #1 was a `push` event at an earlier workflow revision, per the runs API.)
- `.github/workflows/source-preflight.yml:6-7` sets top-level `contents: read`; no job-level override adds scopes. No `packages`, `actions`, `deployments`, `id-token`, or `contents: write` scope exists.
- A grep over `.github/` for `secrets.`, `env:`, `uses:`, `permissions:`, and related terms found no `secrets.*` reference and no `env:` block. The file contains only the three `uses:` lines at `:14`, `:17`, and `:48`.
- The workflow writes only textual inspection output to the job log (`:31-41`) and copies a defconfig plus a commit SHA into an artifact (`:43-45`). Nothing in the definition intentionally writes credential material to logs or artifacts.
- No secret values were requested, read, printed, or stored in this audit. Run log contents were not inspected, so this report does not assert that all historical logs are secret-free; it only reports what the workflow definition instructs.
- GitHub-owned actions at moving `@v4` tags and the external public repository checkout are implicit trust dependencies (`:14,17,48`).

### 3.2 Recommendations for any future pipeline (not implemented)

1. Keep `permissions: {}` at workflow top level and grant `contents: read` only to jobs that check out code. Add `contents: write` only to a separately gated release job; add `id-token: write` only if OIDC-based signing/provenance is actually used.
2. Pin `actions/checkout` and `actions/upload-artifact` (and any future action) to full 40-character commit SHAs, with a comment naming the version. Pin the runner to a specific image version or container digest rather than `ubuntu-latest`.
3. Set `persist-credentials: false` on checkout steps, especially the external source checkout at `.github/workflows/source-preflight.yml:16-22`, unless a later step genuinely needs Git credentials.
4. Do not pass repository secrets to jobs that only compile or inspect source. If future signing/upload credentials are required, use short-lived credentials or OIDC, scope them to a protected environment, and require explicit approval before that job runs.
5. Never echo secret variables, never use `set -x` in steps with secret environment variables, and validate that artifacts/manifests contain no tokens, keys, or environment dumps. Do not rely on log masking as the primary control.
6. Require a named reviewer/protected environment for any release, upload, or distribution step. `workflow_dispatch` alone is not sufficient as a release gate.

---

## 4. Packaging assumptions and risks (existing behavior)

### 4.1 What is and is not present

- There is **no local packaging script** to execute or audit. The local workflow stops after copying the defconfig and source commit (`.github/workflows/source-preflight.yml:43-51`). It does not build a boot image, dtbo image, recovery image, OTA, or flashable package.
- Consequently, no local script encodes an assumption about a device model, partition layout, A/B slots, ramdisk, or recovery behavior. This is a gap, not proof that the target's layout is simple or universal.
- Upstream build definitions only establish kernel image shapes: `Image.gz` plus `dtbs` (`arch/arm64/Makefile:127-132`), appended DTB and overlay flags with specific DT names (`gta7lite_defconfig:438-441`, `arch/arm64/boot/dts/mediatek/Makefile:1-15`). An Android boot header, ramdisk inclusion policy, dtbo partition mapping, and recovery image are not defined locally.
- `arch/arm64/boot/install.sh:37-60` is a host install helper that can execute user-provided `${INSTALLKERNEL}` hooks and write to `$4`; it is not a device packaging script and is not invoked by `.github/workflows/source-preflight.yml`.
- No packaging script was executed during this audit.

### 4.2 Risks for a future packaging stage

| Risk | Basis | Consequence if unaddressed |
|---|---|---|
| Model/variant ambiguity | C1 `REPORT.md:84-100,110-112` records LTE/Wi-Fi variants and a T220-based upstream sync versus the stated SM-T225 target; the workflow only checks `gta7lite_defconfig` (`.github/workflows/source-preflight.yml:24-29`). | A defconfig filename is not proof of target model. Packaging must use separately verified model/firmware evidence before choosing DT overlay or boot assets. |
| Appended-DTB/dtbo assumptions | `gta7lite_defconfig:438-441`; `arch/arm64/boot/dts/mediatek/Makefile:1-15` @ `4497b689`. | If the device firmware does not use the expected appended-DTB layout or dtbo partition, a produced image can be structurally wrong even if it compiles. |
| Ramdisk and recovery policy absent | No local packaging definition; C1 `REPORT.md:64` notes packaging is undefined in this project. | A future script must not silently inherit a ramdisk or recovery image; these need explicit, verified inputs. |
| Slot/A/B and partition layout unverified | No local script; C4 `TASK.md:11` identifies exact model/variant, firmware, partition mapping, stock recovery assets, and rollback evidence as prerequisites. | Wrong slot or partition targeting can make a device unbootable. Device validation is outside C3 ownership. |
| Build-time source mutation | C2 `REPORT.md:67,155-156` records that legacy Kbuild can rewrite vendor/SELinux source files during build. | Packaging a build without capturing the generated source diff would lose reproducibility and review evidence. |
| Toolchain path mismatch | Upstream configs reference `prebuilts/...` paths (`build.config.mtk.aarch64.tiramisu:5,8`; `build.config.mtk.aarch64.gcc:6`) while the pinned root has no `prebuilts/` directory. | A future build may fetch an arbitrary compiler; outputs can differ by compiler version and flags. |
| Signing/key exposure | No signing configuration or secret handling exists locally. | Adding keys to logs, artifacts, or YAML would be a high-impact secret leak; use protected environments/OIDC if signing is ever approved. |

These are risks to be resolved by verified inputs and C4 acceptance criteria; this report does not prescribe device operations or claim that any recovery path exists.

---

## 5. Recommended minimal staged pipeline for a future approved build (recommendations only)

Nothing below exists today and nothing below was created or run. This is a proposal for a later, explicitly approved phase.

| Stage | Purpose | Required inputs / checks | Gate to next stage |
|---|---|---|---|
| 0. Approval and target evidence | Establish authority and device facts before any build. | Explicit user approval; exact model/variant, firmware build, partition/slot/ramdisk/recovery evidence produced by C4; written decision that this is an LTE `gta7lite` target. | Stop if model, firmware, or rollback evidence is missing. C3 does not own device validation. |
| 1. Pin and static checks | Convert all moving inputs to immutable identifiers. | Record project commit; kernel source commit; KSU source commit/tag; action SHAs; runner/container digest; toolchain download URL + checksum; defconfig SHA-256. Run YAML/Actions lint, shell lint for any new scripts, secret-pattern scan, and license/SPDX notice check. | Named reviewer approves the manifest. No build may proceed on `master`, `@v4`, or `ubuntu-latest` alone. |
| 2. Compile-only build | Produce kernel artifacts without packaging or device interaction. | Isolated runner from pinned image; toolchain fetched by checksum; explicit `ARCH=arm64` and defconfig command; capture `.config`, build log, compiler `--version`, generated source diff from any Kbuild rewrites; fail on unexpected dirty tree. | Stop on compile failure or unexplained source rewrites. Artifact remains internal. |
| 3. Artifact metadata and upload | Make the output verifiable. | Upload `Image.gz`, DTB/DTBO outputs, `.config`, build log, source diff, and a `manifest.json` containing all pinned SHAs, toolchain checksum, runner image, action SHAs, command lines, timestamps, and SHA-256 of every output. Set explicit retention. | Automated hash verification plus human review of manifest and diffs. |
| 4. Artifact inspection / static validation | Check shape and safety before release. | Verify manifest hashes; confirm DTB/DTBO names match pinned config; scan artifacts for secrets/tokens; compare expected image shape to the C4 target evidence. Do not infer model compatibility from filenames. | C4 evidence must explicitly match the artifact shape; otherwise stop. |
| 5. Manual release gate | Optional publication of reviewed artifacts. | Protected environment, named approver, release notes explaining exact inputs and unverified device claims. Use OIDC/short-lived credentials only if signing or upload is approved. | Separate explicit approval required before any distribution or device action. |
| 6. Device phase (out of C3 scope) | Only later, user-supervised validation. | Hand off to C4; require verified stock assets and rollback path before any flashing step. | No automatic flashing, deployment, or release-to-device from CI. |

Recommended permission shape for the future pipeline: workflow-level `permissions: {}`; `contents: read` for stages 1–4; `contents: write` only in stage 5 if a GitHub Release is actually created; `id-token: write` only if signed provenance is used. All release credentials must be environment-scoped and never exposed to build jobs.

---

## 6. Gaps and unknowns

1. Build inputs are absent from the repository; no local baseline exists for compiler flags, boot-image format, or packaging commands.
2. The artifact API reports a digest, but artifact contents were not downloaded or independently verified here.
3. Workflow run logs were not inspected; this report describes instructions in the workflow definition, not every historical log line.
4. Upstream root contents were enumerated at the pinned commit (67 entries), but deeper upstream directories were not exhaustively audited; toolchain paths referenced in build configs were checked for absence at root level only.
5. Target identity and recovery/rollback evidence remain unresolved by C1/C4 and must not be inferred from `gta7lite_defconfig` naming alone.
6. No future action, build, release, upload, or device step in section 5 has been implemented or authorized.

## 7. Key evidence index

| Evidence | Lines / fields | Use |
|---|---|---|
| `.github/workflows/source-preflight.yml` | 1, 3-7, 10-11, 13-22, 24-29, 31-45, 47-51 | Workflow identity, dispatch trigger, permissions, runner, checkouts, inspection commands, artifact staging/upload |
| `README.md` | 1-2 | No build documentation |
| Local `find` / `git ls-files -s` | file lists | Only one workflow; no local scripts or packaging files |
| GitHub workflows API | `total_count: 1` | Only one registered workflow |
| GitHub runs/artifacts API | run #1/#2; one artifact + digest + expiry | Observed automation history and artifact handling |
| GitHub releases/tags API | empty arrays | No existing release/tag baseline |
| Upstream `Makefile` @ `4497b689` | 2-5 | Kernel version |
| Upstream `arch/arm64/Makefile` @ `4497b689` | 127-132 | Default `Image.gz` + `dtbs` targets |
| Upstream `arch/arm64/configs/gta7lite_defconfig` @ `4497b689` | 7-11, 22-23, 438-441 | Compiler header, local version, DTB/DTBO selection |
| Upstream `arch/arm64/boot/dts/mediatek/Makefile` @ `4497b689` | 1-15 | DTB/DTBO output expansion |
| Upstream `arch/arm64/boot/install.sh` @ `4497b689` | 37-60 | Host install behavior, not Android packaging |
| Upstream `build.config.mtk.aarch64.tiramisu` @ `4497b689` | 1-18 | LLVM toolchain and absent prebuilt paths |
| Upstream `build.config.mtk.aarch64.gcc` @ `4497b689` | 1-6 | GCC toolchain variant and absent prebuilt path |
| Upstream `build.config.mtk.aarch64` @ `4497b689` | 1-18 | Second LLVM toolchain variant |
| Upstream contents API @ `4497b689` | 67 root entries; no `prebuilts/` | No root toolchain or packaging directory |
| `.mece/cells/C1-SOURCE/REPORT.md` | 15-18, 64, 84-100, 109-115 | Source provenance, variant ambiguity, toolchain conflict, missing packaging baseline |
| `.mece/cells/C2-KSUNEXT/REPORT.md` | 67, 155-156 | Build-time source rewrite and compile-only handoff context |
| `.mece/cells/C4-VALIDATION/TASK.md` | 10-13 | Handoff boundary for validation and rollback evidence |

## 8. Change statement

No project, kernel source, Git history, configuration, workflow, script, task file, build artifact, release, upload, or connected-device state was changed. No GitHub Actions run, build job, release, deployment, packaging script, toolchain, or device command was started or executed. No secret values were requested, accessed, or printed. All inspection was read-only. The only file written for this cell is `.mece/cells/C3-PIPELINE/REPORT.md`.
