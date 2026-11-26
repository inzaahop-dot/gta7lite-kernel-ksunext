# C2-KSUNEXT — KernelSU Next compatibility assessment

Assessment date: 2026-11-26. Scope: read-only compatibility research for the kernel baseline established in `.mece/cells/C1-SOURCE/REPORT.md`. This cell does **not** own source provenance, CI/build-pipeline design, or physical-device validation, and it makes no kernel, workflow, or device changes.

**Bottom line:** The upstream project is `KernelSU-Next/KernelSU-Next`. The C1 baseline (Linux `4.19.191`, `arm64`, vendor Samsung/MediaTek tree, built-in-driver class) is within upstream's advertised kernel range, but it is **not a GKI 2.0 / LKM target** and no method is proven to work on this tree without compilation and runtime evidence. The design-level recommendation for a future approved phase is **built-in `legacy` integration with manual syscall hooks**, because it is the documented non-GKI path with the fewest unverified runtime assumptions. This is a planning choice, not a success claim.

---

## 1. Upstream project, refs, requirements, and source references

### 1.1 Relevant upstream project

| Item | Verified value | Evidence |
|---|---|---|
| Project | `KernelSU-Next/KernelSU-Next` | [repo API](https://api.github.com/repos/KernelSU-Next/KernelSU-Next) |
| Homepage | `https://kernelsu-next.github.io/webpage/` | repo API `homepage` |
| Default branch | `dev` | repo API `default_branch` |
| `dev` HEAD at assessment | `30d9bade7777e3ed8e3cf54c41f1ca141521aaee` | [branches API](https://api.github.com/repos/KernelSU-Next/KernelSU-Next/branches?per_page=100) |
| `legacy` HEAD at assessment | `e5f5ec998de88b9fc43c62cff8f7ae12d3f4800a` | branches API |
| `samsung-lateload` HEAD at assessment | `bad09b19ca337ba35e052df2a7bee788b8bb7c83` | branches API |
| Latest release tag observed | `v3.4.2` → `4c706b5938665866ebe16944a481a7b91e3396af` | [tags API](https://api.github.com/repos/KernelSU-Next/KernelSU-Next/tags?per_page=30) |
| Kernel driver license | GPL-2.0-only for `/kernel`; GPL-3.0-or-later for other files | [dev README](https://github.com/KernelSU-Next/KernelSU-Next/blob/30d9bade7777e3ed8e3cf54c41f1ca141521aaee/docs/README.md#L77-L80) |

KernelSU Next is a fork/derivative line that credits original KernelSU; the compatibility subject here is the Next project and its source, not the original `tiann/KernelSU`.

### 1.2 Documented compatibility

Pinned `dev` documentation states:

- Supported Android kernels: **4.4 through 6.18**.
- `5.10+ (GKI 2.0)`: pre-built images and LKM/KMI.
- `4.19–5.4 (GKI 1.0)`: **KernelSU driver must be built in**.
- `<4.14`: driver required; `3.18+` experimental and may need backports.
- Supported architectures: `arm64-v8a` and `x86_64`; x86_64 has a caution about recent kernels and potential panic.

Source: [dev `docs/README.md:47-60`](https://github.com/KernelSU-Next/KernelSU-Next/blob/30d9bade7777e3ed8e3cf54c41f1ca141521aaee/docs/README.md#L47-L60).

### 1.3 Integration methods documented upstream

1. **GKI 2.0 pre-built / LKM / KMI** — for `5.10+`; installation guide lists release/module flows. Source: [installation guide](https://kernelsu-next.github.io/webpage/pages/installation.html).
2. **Non-GKI automatic kprobe integration** — guide says KernelSU Next uses kprobes for hooks and recommends this when kprobes are reliable; it asks for `CONFIG_KPROBES=y`, `CONFIG_KPROBE_EVENTS=y`, `CONFIG_KSU_KPROBE_HOOKS=y`, `CONFIG_KSU=y`. Source: [non-GKI guide](https://kernelsu-next.github.io/webpage/pages/how-to-integrate-for-non-gki.html).
3. **Non-GKI manual hook integration** — guide says to add `CONFIG_KSU=y` and calls into `do_execve`, `SYSCALL_DEFINE3` in `fs/open.c`, `vfs_read`/`read`, `SYSCALL_DEFINE4` in `fs/stat.c`, and `SYSCALL_DEFINE4` in `kernel/reboot.c`. The same guide notes that kernels older than 5.9 need a `path_umount()` backport in `fs/namespace.c` for the module-umount feature; that is directly relevant to this 4.19.191 baseline. Same source as above.
4. **Legacy source hook modes** — current `legacy` branch adds three explicit choices:
   - `KSU_MANUAL_HOOK`: manual source hooks; depends on `KSU != m`; default `y` when `!KPROBES`.
   - `KSU_KPROBES_HOOK`: depends on `KRETPROBES && KPROBES && HAVE_SYSCALL_TRACEPOINTS`; help text says **“This should not be used on kernel below 5.10.”**
   - `KSU_SYSCALL_TABLE_HOOK`: depends on `ARM64 || X86_64`; help text says it is intended for non-GKI kernels where kprobes are unavailable/non-functional and **requires kernel ≥ 4.17 (pt_regs syscall ABI)**.
   
   Source: [legacy `kernel/Kconfig:44-71`](https://github.com/KernelSU-Next/KernelSU-Next/blob/e5f5ec998de88b9fc43c62cff8f7ae12d3f4800a/kernel/Kconfig#L44-L71).
5. **Samsung late-load compatibility** — the `samsung-lateload` branch contains upstream commit `b8dafae3`, “feat: add Samsung KDP/RKP/DEFEX late-load compatibility (#1507)”. It adds Kconfig switches `KSU_SAMSUNG_KDP`, `KSU_SAMSUNG_RKP`, `KSU_SAMSUNG_DEFEX`, and `KSU_SAMSUNG_NO_PATCH_TEXT`, plus driver compatibility files. Sources: [commit](https://github.com/KernelSU-Next/KernelSU-Next/commit/b8dafae390eba00cba9983e8ef1e1eaad74ba4ed), [pinned Kconfig](https://github.com/KernelSU-Next/KernelSU-Next/blob/bad09b19ca337ba35e052df2a7bee788b8bb7c83/kernel/Kconfig#L46-L76).

### 1.4 Current-source requirements and guidance conflicts

- `dev`/release-style Kconfig: `CONFIG_KSU` is tristate and **depends on `KPROBES && EXT4_FS`**; help text cites `ext4_unregister_sysfs`. Source: [dev `kernel/Kconfig:3-12`](https://github.com/KernelSU-Next/KernelSU-Next/blob/30d9bade7777e3ed8e3cf54c41f1ca141521aaee/kernel/Kconfig#L3-L12). The same dependency shape is present in tag `v3.4.2`: [pinned Kconfig](https://github.com/KernelSU-Next/KernelSU-Next/blob/4c706b5938665866ebe16944a481a7b91e3396af/kernel/Kconfig#L3-L12).
- `legacy` Kconfig does **not** carry the hard `KPROBES && EXT4_FS` dependency and instead selects one of manual / kprobe / syscall-table hooks: [legacy Kconfig](https://github.com/KernelSU-Next/KernelSU-Next/blob/e5f5ec998de88b9fc43c62cff8f7ae12d3f4800a/kernel/Kconfig#L3-L71).
- The published non-GKI guide says `CONFIG_KSU_KPROBE_HOOKS`, while the current `legacy` source uses `CONFIG_KSU_KPROBES_HOOK` and `CONFIG_KSU_SYSCALL_TABLE_HOOK`. These names do not match; treat the guide as partially stale.
- The installation guide fetches `setup.sh` from a `next` ref, but `next` was not present in the branch list or in the `refs/heads/next` / `refs/tags/next` API queries at assessment time; the raw URL did resolve to a `setup.sh` with the same blob SHA as `dev` and `legacy` (`292e2c022e5cbe7515811fc9b9007918c6ef1c95`). This is a ref-resolution/documentation inconsistency, not evidence that any integration works.
- The installation guide explicitly warns that the dev branch “has not been fully tested and may not be suitable for most device environments.”

`setup.sh` behavior is concrete and useful for implementation scoping: it requires `drivers/` or `common/drivers/`, symlinks the KernelSU `kernel/` directory to `drivers/kernelsu`, appends `obj-$(CONFIG_KSU) += kernelsu/` to `drivers/Makefile`, and sources `drivers/kernelsu/Kconfig` from `drivers/Kconfig`. Source: [pinned `kernel/setup.sh`](https://github.com/KernelSU-Next/KernelSU-Next/blob/30d9bade7777e3ed8e3cf54c41f1ca141521aaee/kernel/setup.sh#L13-L63). Separately, the legacy Kbuild conditionally embeds a static preinit runner only when installed UAPI headers exist at `$(objtree)/usr/include/linux/unistd.h` (its comment associates this with `CONFIG_HEADERS_INSTALL`, enabled by `gki_defconfig`); otherwise it sets `-DKSU_NO_PREINIT`. C1 does not record `CONFIG_HEADERS_INSTALL` for `gta7lite_defconfig`, so this is an additional compile-time unknown. Source: [legacy `kernel/Kbuild:69-115`](https://github.com/KernelSU-Next/KernelSU-Next/blob/e5f5ec998de88b9fc43c62cff8f7ae12d3f4800a/kernel/Kbuild#L69-L115).

### 1.5 SELinux / LSM behavior visible in upstream source

- `dev` init calls `ksu_selinux_inode_init()` and aborts initialization if it fails, then calls `ksu_lsm_hook_init()`, `ksu_selinux_hide_init()`, and related features. Source: [dev `kernel/core/init.c:148-164`](https://github.com/KernelSU-Next/KernelSU-Next/blob/30d9bade7777e3ed8e3cf54c41f1ca141521aaee/kernel/core/init.c#L148-L164).
- `dev` `ksu_selinux_inode_init()` uses `selinux_blob_sizes.lbs_inode` via BTF in module builds and otherwise uses `selinux_inode()`; this makes SELinux header/type layout a build-time dependency. Source: [dev `kernel/selinux/inode.c:11-40,66-91`](https://github.com/KernelSU-Next/KernelSU-Next/blob/30d9bade7777e3ed8e3cf54c41f1ca141521aaee/kernel/selinux/inode.c#L11-L40).
- `dev` LSM hooking resolves `security_hook_heads` / static-call tables and live-patches hook slots with `ksu_patch_text()`. Source: [dev `kernel/hook/lsm_hook.c:64-69,87-278,280-390`](https://github.com/KernelSU-Next/KernelSU-Next/blob/30d9bade7777e3ed8e3cf54c41f1ca141521aaee/kernel/hook/lsm_hook.c#L64-L69). That is a newer-kernel-oriented implementation and is not the presumed 4.19 path.
- `legacy` LSM hooking uses `security_add_hooks()` with version-specific signatures; for `4.11+` it uses `security_add_hooks(ksu_hooks, ARRAY_SIZE(ksu_hooks), "ksu")`. Source: [legacy `kernel/hook/lsm_hooks.c:128-155`](https://github.com/KernelSU-Next/KernelSU-Next/blob/e5f5ec998de88b9fc43c62cff8f7ae12d3f4800a/kernel/hook/lsm_hooks.c#L128-L155).
- `legacy` `kernel/Kbuild` performs **build-time source rewrites** and feature detection for older/vendor SELinux: it patches `fs/namespace.c`, `fs/internal.h`, `include/linux/seccomp.h`, `security/selinux/hooks.c`, `security/selinux/selinuxfs.c`, `security/selinux/xfrm.c`, and `security/selinux/include/objsec.h`; it detects `CONFIG_KDP_CRED` in `kernel/cred.c` and `SEC_SELINUX_PORTING_COMMON` in `security/selinux/avc.c`. Source: [legacy `kernel/Kbuild:224-379`](https://github.com/KernelSU-Next/KernelSU-Next/blob/e5f5ec998de88b9fc43c62cff8f7ae12d3f4800a/kernel/Kbuild#L224-L379).

---

## 2. Baseline facts used from C1-SOURCE

All local kernel facts below are taken from `.mece/cells/C1-SOURCE/REPORT.md`; the kernel source itself is not present in this checkout and is fetched by `.github/workflows/source-preflight.yml` at runtime.

| Fact | C1 evidence |
|---|---|
| Local workflow fetches `gta7lite/android_kernel_samsung_ot8` at mutable `master` into `kernel-source` and requires `arch/arm64/configs/gta7lite_defconfig` | `.mece/cells/C1-SOURCE/REPORT.md:24-32` |
| Observed kernel version `4.19.191` (`VERSION=4`, `PATCHLEVEL=19`, `SUBLEVEL=191`) | `C1-SOURCE/REPORT.md:57` |
| `CONFIG_ARM64=y`, `CONFIG_64BIT=y`; `ARCH=arm64` in MTK build config | `C1-SOURCE/REPORT.md:58` |
| Vendor selection: MediaTek `MT6765`, project `ot8`, Samsung/KNOX/TEEGRIS markers | `C1-SOURCE/REPORT.md:59,69-71` |
| `gki_defconfig` exists in the tree, but the workflow selects the vendor-heavy `gta7lite_defconfig` | `C1-SOURCE/REPORT.md:61` |
| `CONFIG_KSU` absent from inspected LTE and Wi-Fi defconfigs | `C1-SOURCE/REPORT.md:74` |
| `CONFIG_KPROBES=y`, `CONFIG_UPROBES=y`, `CONFIG_KRETPROBES=y` already enabled | `C1-SOURCE/REPORT.md:75` |
| Toolchain evidence conflict: generated defconfig records GCC 14.1.1 while MTK build configs set LLVM/Clang; effective toolchain not established | `C1-SOURCE/REPORT.md:65,73,114` |
| Target identity SM-T225 is only partially/conditionally matched; master HEAD is a T220 sync | `C1-SOURCE/REPORT.md:98-104,110-112` |

C1 does **not** record `CONFIG_EXT4_FS`, `CONFIG_KPROBE_EVENTS`, `CONFIG_HAVE_SYSCALL_TRACEPOINTS`, `CONFIG_SECURITY`, `CONFIG_SECURITY_SELINUX`, `CONFIG_SECURITYFS`, `CONFIG_MODULES`, `CONFIG_KDP_CRED`, or `SEC_SELINUX_PORTING_COMMON`. Those are therefore listed below as validation items, not as confirmed facts.

---

## 3. Compatibility matrix

Status meanings used here:

- **Supported** — upstream documents a path applicable to this kernel class; **not** a claim that this target boots or works.
- **Uncertain** — applicable in principle but depends on unverified config, symbol, vendor, or toolchain details.
- **Incompatible** — contradicted by the documented upstream requirement for this baseline.

| Area / approach | Upstream requirement or expectation | C1 baseline evidence | Status | Basis / validation needed |
|---|---|---|---|---|
| Kernel version 4.19.191 | 4.4–6.18 supported; 4.19–5.4 built-in-driver class | `C1-SOURCE/REPORT.md:57` | **Supported** | Version is inside upstream range. Does not imply vendor compatibility. |
| Architecture arm64 / `arm64-v8a` | `arm64-v8a` supported; syscall-table mode depends on `ARM64` | `C1-SOURCE/REPORT.md:58` | **Supported** | Architecture requirement satisfied. |
| Non-GKI built-in driver | Required for 4.19–5.4 | Vendor-heavy selected defconfig; `gki_defconfig` present but unused (`C1-SOURCE/REPORT.md:59,61`) | **Supported** | This is the applicable upstream class. `CONFIG_KSU` must be added (`C1-SOURCE/REPORT.md:74`). |
| GKI 2.0 pre-built image / LKM / KMI | Documented only for 5.10+ | Kernel is 4.19.191 | **Incompatible** | Version-gated by upstream docs. Do not plan a GKI LKM/prebuilt path for this baseline. |
| `dev` / release-style Kconfig path | `CONFIG_KSU` depends on `KPROBES && EXT4_FS` | `KPROBES` confirmed; `EXT4_FS` not in C1 | **Uncertain** | `EXT4_FS` and effective Kconfig must be verified. Also not the documented 4.19 branch. |
| `legacy` manual hook mode | Explicitly documented for fragmented non-GKI kernels; `KSU != m` | `CONFIG_KSU` absent; standard 4.19 VFS/syscall sites expected but not inventoried by C1 | **Supported** | Design-level applicability only; exact function signatures require source inspection and compilation. |
| `legacy` syscall-table hook mode | `ARM64`; kernel ≥ 4.17; intended for non-GKI where kprobes unavailable/non-functional | 4.19.191 arm64 satisfies version/arch; vendor protections unknown | **Uncertain** | Need to verify `sys_call_table` resolution, writability, and Samsung RKP/KDP behavior. Upstream Samsung branch exists precisely because some Samsung kernels block live writes. |
| `legacy` kprobe/tracepoint hook mode | `KRETPROBES && KPROBES && HAVE_SYSCALL_TRACEPOINTS`; source says not below 5.10; guide says kprobes recommended if reliable | `KPROBES`/`KRETPROBES` confirmed; `KPROBE_EVENTS`, `HAVE_SYSCALL_TRACEPOINTS`, and `KSU_KPROBE_HOOKS`/`KSU_KPROBES_HOOK` not in C1 | **Uncertain** | Conflicting guidance: website recommends kprobes for non-GKI, current legacy source discourages below 5.10. Requires targeted compile/runtime validation, not a default choice. |
| Published `CONFIG_KSU_KPROBE_HOOKS` config key | Present in non-GKI guide | Current source uses `KSU_KPROBES_HOOK` and `KSU_SYSCALL_TABLE_HOOK` | **Incompatible** | Exact symbol mismatch; do not copy the guide's config name into implementation. |
| `samsung-lateload` KDP/RKP/DEFEX compatibility | Samsung-specific switches; tied to current `dev` / late-load LKM flow | Samsung/KNOX/TEEGRIS markers present; no C1 evidence of `CONFIG_KDP_CRED`, KDP/RKP/DEFEX, or GKI 2.0 KMI | **Uncertain** | Relevant to vendor family, but no evidence it applies to a built-in 4.19 driver. Treat as a research lead, not a validated method. |
| SELinux/LSM integration | Requires LSM hooks and SELinux internals/headers; legacy Kbuild rewrites vendor SELinux files as needed | C1 records no `CONFIG_SECURITY`, `CONFIG_SECURITY_SELINUX`, `CONFIG_SECURITYFS`, or SELinux file contents | **Uncertain** | Vendor SELinux divergence is a primary compile/runtime risk. Must inspect the pinned source and generated `.config`. |
| Manager signature / policy integration | Kconfig offers `KSU_DISABLE_MANAGER`, `KSU_DISABLE_POLICY`; manager signature hash/size compiled into Kbuild | No local evidence about manager version | **Uncertain** | Out of scope for compatibility proof; validate only in a later integration phase. |
| Toolchain | Upstream Kbuild uses host `$(CC)` checks and kernel build flags; current release LKMs target GKI KMIs | GCC 14.1.1 recorded vs LLVM/Clang build configs; effective toolchain unresolved | **Uncertain** | Compile with the actual selected toolchain; do not infer success from either recorded clue. |
| UAPI headers / embedded preinit runner | Legacy Kbuild builds `preinit/runner.elf` only when `$(objtree)/usr/include/linux/unistd.h` exists; otherwise `-DKSU_NO_PREINIT` | C1 does not record `CONFIG_HEADERS_INSTALL` for `gta7lite_defconfig` | **Uncertain** | Verify header installation in the effective build path; feature loss or build failure must be observed, not assumed. |
| Target identity / exact firmware | Upstream compatibility is kernel-class based, not SM-T225 certified | C1: partial LTE match, T220 sync, no SM-T225 string | **Uncertain** | Device/model identity remains unresolved by C1. |

**Confirmed incompatibilities are limited to:** (1) the GKI 2.0 prebuilt/LKM/KMI path for a 4.19 baseline, and (2) the guide's `CONFIG_KSU_KPROBE_HOOKS` key versus current source symbols. Everything else above is supported-in-class or needs compilation/runtime evidence.

---

## 4. Candidate integration points

### 4.1 Driver wiring (common to built-in approaches)

- Add the upstream `kernel/` payload under the fetched kernel tree as `kernel-source/drivers/kernelsu` (symlink or vendored copy) using the documented `setup.sh` behavior.
- Wire `kernel-source/drivers/Makefile` and `kernel-source/drivers/Kconfig`.
- Enable `CONFIG_KSU=y` in `kernel-source/arch/arm64/configs/gta7lite_defconfig`. The Wi-Fi variant `kernel-source/arch/arm64/configs/gta7litewifi_defconfig` is a separate C1-known config and should only be considered if that variant is explicitly in scope.
- For manual hooks, keep `CONFIG_KSU` built in (`y`), not `m`: `KSU_MANUAL_HOOK` depends on `KSU != m` in the legacy source.

### 4.2 Hook selection

1. **Manual hooks (preferred design-level candidate for this target).** Add guarded calls in the documented standard sites:
   - `kernel-source/fs/exec.c` → `ksu_handle_execveat(...)` in `do_execve` and `compat_do_execve`.
   - `kernel-source/fs/open.c` → `ksu_handle_faccessat(...)` in `faccessat`.
   - `kernel-source/fs/read_write.c` → `ksu_handle_sys_read(...)` in `read`.
   - `kernel-source/fs/stat.c` → `ksu_handle_stat(...)` in `newfstatat`.
   - `kernel-source/kernel/reboot.c` → `ksu_handle_sys_reboot(...)` in `sys_reboot`.

   Upstream patch text: [non-GKI guide](https://kernelsu-next.github.io/webpage/pages/how-to-integrate-for-non-gki.html).

2. **Syscall-table hooks (secondary candidate).** No manual call sites in core kernel files; relies on `kernel-source/kernel/hook/syscall_table_hook.c` and `kernel-source/kernel/hook/patch_memory.c` in the legacy driver. Applicable by version/arch but risky on Samsung devices with RKP-style write protection.

3. **Kprobe/tracepoint hooks (validation-only candidate).** Uses `kernel-source/kernel/hook/hook_manager.c` in the legacy driver. Baseline already has `KPROBES`/`KRETPROBES`, but the source warns against use below 5.10 and the website/source config names conflict.

4. **Samsung late-load branch (research lead only).** Relevant files in upstream are `kernel/compat/samsung_kdp.c`, `kernel/compat/samsung_defex.c`, `kernel/compat/samsung_rkp.c`, and modified `kernel/hook/arm64/patch_memory.c` / `kernel/hook/arm64/syscall_hook.c` / `kernel/hook/syscall_hook_manager.c` / `kernel/feature/selinux_hide.c` / `kernel/policy/app_profile.c`. See [commit `b8dafae3`](https://github.com/KernelSU-Next/KernelSU-Next/commit/b8dafae390eba00cba9983e8ef1e1eaad74ba4ed). No evidence in C1 makes this the right path for 4.19 built-in integration.

### 4.3 SELinux / LSM integration points

- Target-side: `kernel-source/security/selinux/hooks.c`, `kernel-source/security/selinux/selinuxfs.c`, `kernel-source/security/selinux/xfrm.c`, `kernel-source/security/selinux/include/objsec.h`, `kernel-source/security/selinux/ss/services.c`, and `kernel-source/security/selinux/avc.c` are the files upstream either rewrites, detects, or depends on in the legacy path.
- Driver-side: `kernel-source/drivers/kernelsu/kernel/hook/lsm_hooks.c`, `kernel-source/drivers/kernelsu/kernel/selinux/selinux.c`, `rules.c`, `sepolicy.c`, `inode.c`, and `kernel-source/drivers/kernelsu/kernel/feature/selinux_hide.c`.
- Samsung-specific markers to verify, not assume: `CONFIG_KDP_CRED` in `kernel-source/kernel/cred.c` and `SEC_SELINUX_PORTING_COMMON` in `kernel-source/security/selinux/avc.c`.

### 4.4 Build/toolchain coupling

- The effective build path in this project is not established by C1: generated defconfig says GCC 14.1.1, while `build.config.mtk.aarch64.tiramisu` sets LLVM/Clang. Any integration must be compiled with the actual selected toolchain before claims are made.
- Legacy Kbuild's `sed`-based backports mutate target kernel source during build; this makes build reproducibility and review of generated diffs a compatibility concern, not merely a patch-application detail.

---

## 5. Design-level comparison and recommendation

| Approach | Advantages for this baseline | Disadvantages / unknowns | Design verdict |
|---|---|---|---|
| GKI 2.0 LKM / prebuilt | No core kernel hook edits when applicable | Upstream restricts to 5.10+; target is 4.19.191 built-in class; no packaging baseline in C1 | **Reject for this baseline** |
| Built-in `legacy` + manual hooks | Documented non-GKI path; avoids kprobe stability questions and syscall-table write restrictions; works with `KSU=y` | Requires edits in five standard kernel files; vendor signatures may differ; LSM/SELinux backports still need compile validation | **Recommended planning default** |
| Built-in `legacy` + syscall-table hooks | Version/arch criteria are met; upstream explicitly targets non-GKI where kprobes fail | Live patching of `sys_call_table`; Samsung RKP/KDP may block writes; symbol resolution risk | **Secondary candidate**, only after symbol/protection inspection |
| Built-in `legacy` + kprobe/tracepoint hooks | Baseline already enables `KPROBES` and `KRETPROBES`; website recommends kprobes when reliable | Current source says not below 5.10; `HAVE_SYSCALL_TRACEPOINTS` and correct config name unverified; bootloop is a documented failure mode | **Validation-only**, not a default |
| `dev`/stable-style Kconfig with kprobes | Simpler Kconfig surface | Hard `KPROBES && EXT4_FS` dependency; dev warned untested; not the documented 4.19 non-GKI branch | **Do not default** |
| `samsung-lateload` | Directly addresses Samsung KDP/RKP/DEFEX on current upstream | Tied to dev/late-load LKM context; no C1 evidence for these mechanisms on this 4.19 tree | **Research lead only** |

**Recommendation:** For a future separately approved implementation phase, start from the pinned `legacy` branch and evaluate **built-in `CONFIG_KSU=y` with `CONFIG_KSU_MANUAL_HOOK=y`** as the primary design. Keep syscall-table mode as a fallback only if source inspection confirms the required symbols and no vendor write protection; do not select kprobe mode or GKI LKM as the default. Confidence is **moderate at design level only**; no build or runtime success is claimed.

---

## 6. Risks, unknowns, and recommended next phase

### Risks / unknowns

1. **Unpinned kernel source:** C1 records that CI tracks `master`, not a commit (`C1-SOURCE/REPORT.md:32,109`). Any hook signature or SELinux backport assessment can change with a source update.
2. **Config gap:** `CONFIG_KSU` is absent (`C1-SOURCE/REPORT.md:74`), and C1 does not record `EXT4_FS`, `KPROBE_EVENTS`, `HAVE_SYSCALL_TRACEPOINTS`, SELinux/LSM configs, or Samsung KDP/RKP markers.
3. **Guidance drift:** Website config names and refs disagree with current source (`KSU_KPROBE_HOOKS` vs `KSU_KPROBES_HOOK`; `next`/`stable` refs not present in queried ref lists).
4. **Kprobe reliability:** Upstream's own non-GKI guide describes bootloop as a symptom of broken kprobes, while current legacy Kconfig discourages kprobe mode below 5.10.
5. **Samsung/MediaTek hardening:** KNOX, TEEGRIS, MTK extensions, and possible KDP/RKP/DEFEX behavior may affect credential changes, syscall-table writes, and SELinux policy manipulation. C1 records vendor markers but not these exact mechanisms.
6. **SELinux/LSM divergence:** Legacy Kbuild rewrites SELinux files based on `grep` checks; vendor-modified files may not match expected patterns, producing either build failure or incorrect runtime behavior.
7. **Toolchain inconsistency:** GCC vs LLVM evidence conflict remains unresolved (`C1-SOURCE/REPORT.md:65,73,114`). The legacy Kbuild's optional preinit runner also depends on installed UAPI headers that C1 does not record, so its behavior on this build path is unknown.
8. **Target identity:** C1's SM-T225 conclusion is partial/conditional and notes a T220-based sync (`C1-SOURCE/REPORT.md:98-104`).

### Recommended next phase (for separate approval; not performed here)

1. Pin and record the exact kernel source commit (owner: source/C1 and pipeline/C3, not C2) and capture the full generated `.config`.
2. Verify the unknown config symbols and the exact text of the five manual-hook functions plus the SELinux files listed above.
3. In an isolated, approved implementation branch only: vendor the pinned KernelSU Next `legacy` source, wire `drivers/`, enable `CONFIG_KSU=y` and manual hook mode in `gta7lite_defconfig`, and review every Kbuild-generated source diff before compiling.
4. Perform compile-only validation with the effective toolchain. Do not flash or interact with a device.
5. Hand build artifacts and observed diagnostics to C4 for its non-destructive validation matrix; C2 does not own device testing.
6. Only if manual mode is blocked by vendor code, evaluate syscall-table mode with explicit evidence of writable/resolvable `sys_call_table`; evaluate Samsung late-load separately and only with evidence that its KDP/RKP/DEFEX assumptions apply.

---

## 7. Files likely to require changes in a future, separately approved implementation phase

These are **candidates for review/change only**. None was modified by this cell. Paths are relative to the kernel tree that `.github/workflows/source-preflight.yml` checks out as `kernel-source` (C1 `:16-22`); prefix with `kernel-source/` when applied.

### 7.1 Direct integration sites

| Path | Likely change |
|---|---|
| `drivers/Makefile` | Add `obj-$(CONFIG_KSU) += kernelsu/` (per `setup.sh`). |
| `drivers/Kconfig` | Source `drivers/kernelsu/Kconfig` (per `setup.sh`). |
| `arch/arm64/configs/gta7lite_defconfig` | Add `CONFIG_KSU=y` and selected hook-mode symbol (`CONFIG_KSU_MANUAL_HOOK=y` under the recommendation). |
| `fs/exec.c` | Add guarded `ksu_handle_execveat(...)` calls in `do_execve` / `compat_do_execve`. |
| `fs/open.c` | Add guarded `ksu_handle_faccessat(...)` call. |
| `fs/read_write.c` | Add guarded `ksu_handle_sys_read(...)` call. |
| `fs/stat.c` | Add guarded `ksu_handle_stat(...)` call. |
| `kernel/reboot.c` | Add guarded `ksu_handle_sys_reboot(...)` call; legacy Kbuild uses this file to detect manual-hook presence. |

### 7.2 Build-time rewrite / conflict-review sites (legacy Kbuild)

| Path | Why |
|---|---|
| `fs/namespace.c` | Kbuild may inject `can_umount()` / `path_umount()` backports. |
| `fs/internal.h` | Kbuild may declare `path_umount()`. |
| `include/linux/seccomp.h` | Kbuild may add `filter_count` to `struct seccomp`. |
| `security/selinux/hooks.c` | Kbuild may replace `inode->i_security` / `cred->security` accesses with `selinux_inode()` / `selinux_cred()`. |
| `security/selinux/selinuxfs.c` | Kbuild may rewrite `selinux_inode()` access. |
| `security/selinux/xfrm.c` | Kbuild may rewrite `selinux_cred()` access. |
| `security/selinux/include/objsec.h` | Kbuild may insert `selinux_inode()` / `selinux_cred()` helpers. |

### 7.3 Inspection-only vendor markers

| Path | Why |
|---|---|
| `kernel/cred.c` | Legacy Kbuild checks for `CONFIG_KDP_CRED` and defines `SAMSUNG_UH_DRIVER_EXIST`. |
| `security/selinux/avc.c` | Legacy Kbuild checks for `SEC_SELINUX_PORTING_COMMON` and defines `SAMSUNG_SELINUX_PORTING`. |
| `security/selinux/ss/services.c` | SELinux policy/RCU semantics used by KernelSU rules; verify before assuming backport compatibility. |

### 7.4 Optional variant

| Path | Condition |
|---|---|
| `arch/arm64/configs/gta7litewifi_defconfig` | Only if the Wi-Fi variant becomes explicitly in scope; C1 says the workflow selects the LTE config (`:61,:88-89`). |

C3-owned workflow/packaging files are intentionally not listed as C2 change targets.

---

## 8. Evidence index

### Local / project

| Path | Lines | Use |
|---|---|---|
| `.mece/cells/C1-SOURCE/REPORT.md` | 24-32, 57-75, 98-104, 109, 114 | Baseline kernel version, architecture, config, vendor markers, kprobes, unpinned source, toolchain conflict |
| `.github/workflows/source-preflight.yml` (as described by C1) | 16-22, 24-29, 31-45 | Runtime fetch into `kernel-source`; LTE defconfig selection; no committed source snapshot |
| `.mece/WAVE.md` | 26-31 | C2 ownership boundary |

### Upstream KernelSU Next

| URL / ref | Evidence |
|---|---|
| [repo API](https://api.github.com/repos/KernelSU-Next/KernelSU-Next) | Project identity, default branch `dev`, homepage, license metadata |
| [branches API](https://api.github.com/repos/KernelSU-Next/KernelSU-Next/branches?per_page=100) | `dev`, `legacy`, `samsung-lateload` HEAD SHAs; no `next`/`stable` branch in response |
| [tags API](https://api.github.com/repos/KernelSU-Next/KernelSU-Next/tags?per_page=30) | `v3.4.2`, `v3.4.0-legacy` and commit SHAs |
| [dev `docs/README.md:47-60`](https://github.com/KernelSU-Next/KernelSU-Next/blob/30d9bade7777e3ed8e3cf54c41f1ca141521aaee/docs/README.md#L47-L60) | Supported kernel ranges and architectures |
| [installation guide](https://kernelsu-next.github.io/webpage/pages/installation.html) | Setup modes; dev-branch warning; `next` ref in commands |
| [non-GKI integration guide](https://kernelsu-next.github.io/webpage/pages/how-to-integrate-for-non-gki.html) | Automatic kprobe and manual hook instructions; five hook call sites |
| [dev `kernel/Kconfig:3-12`](https://github.com/KernelSU-Next/KernelSU-Next/blob/30d9bade7777e3ed8e3cf54c41f1ca141521aaee/kernel/Kconfig#L3-L12) | `KSU` tristate, `KPROBES && EXT4_FS` dependency |
| [legacy `kernel/Kconfig:44-71`](https://github.com/KernelSU-Next/KernelSU-Next/blob/e5f5ec998de88b9fc43c62cff8f7ae12d3f4800a/kernel/Kconfig#L44-L71) | Manual / kprobe / syscall-table hook modes and constraints |
| [legacy `kernel/Kbuild:192-221`](https://github.com/KernelSU-Next/KernelSU-Next/blob/e5f5ec998de88b9fc43c62cff8f7ae12d3f4800a/kernel/Kbuild#L192-L221) | Hook-mode compile definitions and manual-hook detection |
| [legacy `kernel/Kbuild:224-379`](https://github.com/KernelSU-Next/KernelSU-Next/blob/e5f5ec998de88b9fc43c62cff8f7ae12d3f4800a/kernel/Kbuild#L224-L379) | Backports, SELinux rewrites, Samsung detection |
| [legacy `kernel/Kbuild:69-115`](https://github.com/KernelSU-Next/KernelSU-Next/blob/e5f5ec998de88b9fc43c62cff8f7ae12d3f4800a/kernel/Kbuild#L69-L115) | Conditional preinit runner and UAPI-header dependency |
| [legacy `kernel/hook/lsm_hooks.c:128-155`](https://github.com/KernelSU-Next/KernelSU-Next/blob/e5f5ec998de88b9fc43c62cff8f7ae12d3f4800a/kernel/hook/lsm_hooks.c#L128-L155) | Version-specific `security_add_hooks()` use |
| [dev `kernel/core/init.c:148-164`](https://github.com/KernelSU-Next/KernelSU-Next/blob/30d9bade7777e3ed8e3cf54c41f1ca141521aaee/kernel/core/init.c#L148-L164) | SELinux inode init and LSM hook init ordering |
| [dev `kernel/selinux/inode.c:11-40,66-91`](https://github.com/KernelSU-Next/KernelSU-Next/blob/30d9bade7777e3ed8e3cf54c41f1ca141521aaee/kernel/selinux/inode.c#L11-L40) | SELinux blob/BTF dependency |
| [dev `kernel/hook/lsm_hook.c`](https://github.com/KernelSU-Next/KernelSU-Next/blob/30d9bade7777e3ed8e3cf54c41f1ca141521aaee/kernel/hook/lsm_hook.c#L64-L69) | Live LSM hook slot patching and static-call handling |
| [Samsung late-load commit](https://github.com/KernelSU-Next/KernelSU-Next/commit/b8dafae390eba00cba9983e8ef1e1eaad74ba4ed) | KDP/RKP/DEFEX compatibility scope |
| [pinned `kernel/setup.sh`](https://github.com/KernelSU-Next/KernelSU-Next/blob/30d9bade7777e3ed8e3cf54c41f1ca141521aaee/kernel/setup.sh#L13-L63) | Exact `drivers/` wiring performed by setup script |

---

## 9. Change statement

No project, kernel source, Git history, configuration, workflow, task file, build artifact, or connected-device state was changed. No KernelSU component was built, installed, or executed. No root, ADB, fastboot, Odin, flashing, or other device command was run. All repository and upstream inspection was read-only. The only file written for this cell is `.mece/cells/C2-KSUNEXT/REPORT.md`.
