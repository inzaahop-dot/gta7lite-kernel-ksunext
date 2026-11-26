# C2-KSUNEXT — KernelSU Next compatibility assessment

## Exclusive ownership
Analyze KernelSU Next upstream compatibility and potential kernel integration points. Do not own source provenance, CI/build-pipeline design, or physical-device validation.

## Inputs
Use the repository baseline established in `C1-SOURCE/REPORT.md`. If it is absent, limit work to clearly labeled preliminary observations and do not guess target-specific integration details.

## Tasks
1. Identify the relevant KernelSU Next upstream project, supported integration methods, documented requirements, and current upstream guidance.
2. Compare those requirements with the verified kernel version, architecture, configuration, and vendor modifications.
3. Identify likely integration points and compatibility risks (including relevant hooks, SELinux behavior, LSM configuration, GKI/non-GKI status, and toolchain constraints where applicable).
4. Compare supported integration approaches at a design level; recommend one only when evidence supports it.
5. List exact source files likely to require changes in a future, separately approved implementation phase. Do not make those changes.

## Evidence standard
Use upstream documentation/source links and exact local paths. Distinguish confirmed incompatibilities from items requiring compilation or runtime validation.

## Boundaries
- Read-only analysis; no patches or source edits.
- Do not build, install, or execute KernelSU components.
- Do not run root, ADB, fastboot, Odin, or device commands.
- Do not claim that a method works without build/runtime evidence.

## Deliverable
Write only `.mece/cells/C2-KSUNEXT/REPORT.md` with:
- upstream requirements and source references
- compatibility matrix (supported / uncertain / incompatible)
- candidate integration points
- risks, unknowns, and recommended next phase
- explicit statement that no project or device state was changed
