# C4-VALIDATION — Non-destructive validation and recovery plan

## Exclusive ownership
Define future acceptance criteria, non-destructive checks, and recovery planning. Do not audit source provenance, design KSU integration, or change CI.

## Inputs
Use `C1-SOURCE`, `C2-KSUNEXT`, and `C3-PIPELINE` reports. If any are missing, identify the resulting gaps rather than filling them with assumptions.

## Tasks
1. Build a staged validation matrix: static/source checks, compile-time checks, artifact inspection, and only later user-supervised device testing.
2. Define evidence required before any device test is proposed (exact model/variant, current firmware/build, boot image/partition mapping, verified stock recovery assets, and known-good rollback path).
3. Specify acceptance criteria for boot stability, SELinux state, root functionality, logging, and regression checks without prescribing risky device operations.
4. Identify stop conditions and rollback decision points. Any eventual flashing/unlocking action must require a separate explicit user approval and model-specific verified instructions.
5. List what cannot be validated without physical-device access.

## Boundaries
- Planning only; no device interaction or commands.
- Do not instruct the user to unlock, flash, wipe, or alter partitions in this wave.
- Do not claim a recovery path is available unless its artifacts and compatibility are verified.
- Do not modify project files other than this cell's report.

## Deliverable
Write only `.mece/cells/C4-VALIDATION/REPORT.md` with:
- staged validation matrix and acceptance criteria
- prerequisites and stop conditions
- recovery evidence gaps
- open questions requiring the user
- explicit statement that no project or device state was changed
