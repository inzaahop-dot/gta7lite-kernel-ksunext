# C1-SOURCE — Source and target baseline audit

## Exclusive ownership
Audit source provenance and target identity only. Do not design KernelSU Next integration, alter CI, or create a device test plan.

## Tasks
1. Inspect the repository's current branch, commit, top-level layout, submodules, and source acquisition workflow.
2. Identify the exact kernel source repository, upstream base/version, relevant device tree(s), defconfig(s), architecture, and build target(s).
3. Verify whether the source and configuration actually match Samsung Galaxy Tab A7 Lite LTE `SM-T225` / MT8768T. Distinguish LTE and Wi-Fi variants and flag any ambiguity.
4. Record relevant kernel version, Android compatibility clues, vendor-specific modifications, and licensing notices.
5. Identify missing or contradictory baseline information. Do not infer compatibility from a filename alone.

## Evidence standard
Cite repository paths and line numbers where possible, plus exact commit/ref identifiers and upstream URLs. Separate verified facts from hypotheses.

## Boundaries
- Read-only inspection.
- Do not edit source, configs, workflows, docs, or build scripts.
- Do not download or execute untrusted binaries/scripts.
- Do not run builds or interact with a connected device.

## Deliverable
Write only `.mece/cells/C1-SOURCE/REPORT.md` with:
- verified repository/source baseline
- target-device match assessment
- relevant paths and evidence
- uncertainties and blockers
- explicit statement that no project or device state was changed
