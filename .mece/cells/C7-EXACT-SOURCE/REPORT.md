# C7 — Exact T225 BEYE4 source assembly and build-input audit

Audit date: 2026-11-28. Target: Samsung Galaxy Tab A7 Lite LTE, SM-T225, firmware lineage `T225XXSBEYE4`. Scope: read-only source assembly and build-input inspection in GitHub Actions. No KernelSU patch, kernel compile, package, signing, release, or device operation is performed by this cell.

## 1. Source provenance and assembly

Samsung OSRC release `13155` associates `T225XXSBEYE4` with:
- Base archive: `SM-T220_14_Opensource.zip`
- Target delta: `SM-T225_T225N_T227_14_Opensource_dts.zip`

The T225 archive instructions explicitly say to unpack the T220 `T220XXSAEYE4` source first, then apply the T225 BEYE4 update. GitHub Actions run [46153618068](https://github.com/inzaahop-dot/gta7lite-kernel-ksunext/actions/runs/46153618068) successfully downloaded both pinned mirror archives, verified SHA-256, extracted the T220 kernel tree, overlaid the T225 delta, and confirmed the expected LTE files exist.

| Archive | SHA-256 |
|---|---|
| `SM-T220_14_Opensource.zip` | `5bed82eb953cd6b726a6776e161bac1f904ec8c45afdb42ed2ac3648aaa9e806` |
| `SM-T225_T225N_T227_14_Opensource_dts.zip` | `a4597bf68e43a6c9462ac775f2980b39d8cb6b4fd95b6bc99ef87f9c29ab3634` |

**Trust boundary:** OSRC release metadata is Samsung evidence; archive bytes were retrieved from the Internet Archive mirror because direct OSRC downloads were unavailable during C6. The hashes are mirror-integrity pins, not Samsung-authenticated signatures.

The assembled source reports Linux `4.19.191`. The T225 delta contains only:
- `arch/arm64/boot/dts/mediatek/Makefile`
- `arch/arm64/configs/gta7lite_defconfig`
- `arch/arm64/configs/gta7lite_00_defconfig`

It contains no DTS/DTSI files. The T225 Makefile delta selects the LTE `mt8768t_gta7lite_eur_open_00.dtb` target.

## 2. Correct build entry point

The T225-specific `T225XXSBEYE4_kernel.txt` instructs:
- `ANDROID_MAJOR_VERSION=r`
- `ARCH=arm64`
- `gta7lite_defconfig`
- `make` with `KCFLAGS=-w CONFIG_SECTION_MISMATCH_WARN_ONLY=y`
- Output kernel: `arch/arm64/boot/Image.gz`

**Do not use the bundled `build_kernel.sh` unchanged for the LTE target.** It defaults to `gta7litewifi_defconfig`, a Wi-Fi configuration, and uses a separate GCC 4.9 toolchain path. The T225-specific instructions select the LTE defconfig and Clang r383902.

The assembled source tree itself does not contain the Clang toolchain. A separate CI preflight is testing the AOSP Gitiles `clang-r383902` directory pinned at commit `0e9e7035bf8ad42437c6156e5950eab13655b26c`. The toolchain preflight [run 46154239720](https://github.com/inzaahop-dot/gta7lite-kernel-ksunext/actions/runs/46154239720) completed successfully and reported:
- Android Clang `11.0.1` (6443078 based on r383902)
- LLD `11.0.1`
- AOSP Gitiles commit `0e9e7035bf8ad42437c6156e5950eab13655b26c`, directory tree `34feae7abc38fffe258af1842a4c19746368cec6`

The Gitiles-generated tarball SHA-256 varied between downloads from the same immutable commit, so the build pins the Gitiles commit and verifies the compiler version rather than treating the generated compressed archive hash as a stable release checksum. The Clang directory does not include `aarch64-linux-gnu-*` binutils; the compile-only workflow installs AArch64 GNU binutils separately and places those tools under the documented cross-prefix. This is recorded as a toolchain deviation to review against Samsung's build environment.

## 3. KernelSU Next source pin and manual-hook adaptation

Pinned upstream source: `KernelSU-Next/KernelSU-Next`, legacy branch commit `e5f5ec998de88b9fc43c62cff8f7ae12d3f4800a`.

The pinned legacy Kconfig provides `CONFIG_KSU_MANUAL_HOOK`; Kbuild accepts manual mode only when the target `kernel/reboot.c` contains the `ksu_handle_sys_reboot` call site. The source inspection confirms the corresponding implementation/API is present in the pinned KSU tree:
- `ksu_handle_execveat(...)`: `kernel/core/init.c`
- `ksu_handle_faccessat(...)` and `ksu_handle_stat(...)`: `kernel/feature/sucompat.c`
- `ksu_handle_sys_read(unsigned int fd)`: `kernel/runtime/ksud_integration.c`
- `ksu_handle_sys_reboot(...)`: `kernel/supercall/supercall.c`

The current public non-GKI guide's sample read-hook call has a different signature from the pinned legacy source. Therefore the guide patch must **not** be copied verbatim: the pinned implementation takes only the file descriptor for `ksu_handle_sys_read`. The five target-side call-site candidates identified in the assembled LTE tree are:
- `fs/exec.c`: `do_execve` and `compat_do_execve`
- `fs/open.c`: `SYSCALL_DEFINE3(faccessat)`
- `fs/read_write.c`: `SYSCALL_DEFINE3(read)`
- `fs/stat.c`: `SYSCALL_DEFINE4(newfstatat)`
- `kernel/reboot.c`: `SYSCALL_DEFINE4(reboot)`

No hook edits have been applied yet. Exact context-sensitive patching and compilation are the next validation steps.

## 4. Current verdict

**GO for a controlled compile-only integration experiment; NO-GO for flashing or claiming a bootable kernel.**

Established:
- Exact Samsung OSRC release association and required base-plus-delta sequence.
- Successful extraction and assembly of the pinned mirror archives in CI.
- Correct LTE defconfig and kernel build target from the T225-specific instructions.
- Pinned KernelSU Next legacy source and candidate manual-hook API.

Still unverified:
- AOSP Clang archive availability and actual compiler version on the runner.
- Whether the complete unmodified T225 source builds with the prescribed toolchain.
- Whether the five manual-hook patches compile cleanly with this Samsung 4.19.191 tree.
- Runtime bootability, device-tree/boot-image integration, and recovery path.

No device command, flash, unlock, wipe, reboot, partition operation, or firmware modification was performed.
