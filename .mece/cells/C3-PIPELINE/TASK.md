# C3-PIPELINE — Build and artifact pipeline audit

## Exclusive ownership
Inspect existing automation, toolchain declarations, build scripts, packaging, and artifact handling. Do not own kernel-source/KSU integration design or physical-device validation.

## Tasks
1. Inventory existing GitHub Actions workflows and scripts related to source checkout, toolchain setup, kernel builds, packaging, and artifact upload.
2. Determine whether builds are reproducible from pinned inputs; identify unpinned actions, moving branches/tags, implicit dependencies, and missing provenance.
3. Review permissions and secret usage. Recommend least-privilege permissions and ensure no secret values are exposed in logs or reports.
4. Inspect whether packaging scripts make assumptions about device model, partition layout, slots, ramdisk, or recovery behavior. Do not execute packaging scripts.
5. Propose a minimal staged pipeline for a future approved build, including static checks, build, artifact metadata, and manual release gates.

## Evidence standard
Cite exact workflow/script paths and line numbers. Clearly separate existing behavior from recommendations.

## Boundaries
- Read-only inspection; do not edit workflows or scripts.
- Do not start GitHub Actions, build jobs, releases, uploads, or deployments.
- Do not read or print secret values.
- Do not execute repository scripts or downloaded toolchains.

## Deliverable
Write only `.mece/cells/C3-PIPELINE/REPORT.md` with:
- workflow and script inventory
- reproducibility/security findings
- packaging assumptions and risks
- recommended future pipeline stages and gates
- explicit statement that no project or device state was changed
