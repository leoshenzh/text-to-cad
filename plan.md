# text-to-cad runtime plan

## Objective

Maintain Leo's six installed CAD skills from one governed repository, with CAD
generation, Explorer preview, and DWG intake exposed through the single `cad`
entry point.

## Authority and branches

- Upstream: `earthtojake/text-to-cad`.
- Leo fork: `leoshenzh/text-to-cad`.
- Fork `main`: tracks the current upstream release and is not overwritten by the
  older local customization.
- Runtime branch: `leo/cad-consolidated-legacy`.
- Runtime skill entries: `cad`, `sdf`, `urdf`, `srdf`, `step-parts`, and
  `sendcutsend`, linked into both `~/.claude/skills` and `~/.codex/skills`.

## Current state

- The runtime branch consolidates the former `cad-explorer` skill into
  `skills/cad/scripts/explorer`.
- Its base is upstream commit `f921990e` from 2026-05-14; the consolidation
  commit is `d8c83197` from 2026-07-31.
- The current upstream main line is substantially newer. Upgrading the runtime
  therefore requires a separate migration and acceptance pass; it must not be
  presented as a routine fast-forward.

## Milestones

- [x] Preserve the pre-cleanup consolidation commit with a local and remote tag.
- [x] Create Leo's fork without replacing its current upstream-based main line.
- [x] Align runtime docs, prompts, install paths, and tests with the unified CAD
  entry point.
- [x] Run structural and dependency-available test suites.
- [x] Publish the runtime branch and verify the remote readback.
- [x] Move the canonical checkout out of the Claude scan tree and relink both
  agents.
- [x] Verify both agent startup inventories and all six runtime skill links.

## Verification snapshot

- All six skill definitions pass the structural validator.
- The dependency-free Explorer suite passes 112 tests; MoveIt2 passes 17 tests;
  the inspect wrapper passes 4 tests.
- The full Explorer suite reaches 121 of 126 passing tests. The remaining five
  require the uninstalled `three` and `vite` packages; no dependency download
  was authorized for this maintenance task.
- Codex reports 171 skills, with the six runtime entries and both core quality
  skills present exactly once.
- Claude reports 99 user skills plus 2 plugin skills. Its plugin loader reports
  4 enabled, 16 disabled, and zero plugin errors; no text-to-cad circular link
  remains.
- The current DWG bridge is
  `/Volumes/docker/FadiorProductSystem/source/quote/scripts/mini_qcad_dwg.sh`.
  Mac Studio invokes it, but Mac mini QCAD performs the actual DWG conversion.
  A 2026-08-27 live smoke converted the 759,424-byte Milan DWG into a valid
  one-page, 3,036-byte PDF with QCAD 3.32.9 on mini.
- The old `Fadiorteam/projects/tools/quote-tool/` path was removed during the
  2026-08-13 ProductSystem migration and must not be used.
- Modeling is not currently runnable on Studio or mini because the required
  `build123d` / `OCP` environment is absent. Explorer source is present, but
  its runtime dependencies are absent on both machines. Skill presence and
  source synchronization are therefore not evidence that those two paths run.

## Constraints and risks

- Do not install CAD or Explorer dependencies without separate approval if the
  download may exceed 30 MB.
- The CAD skill must use the ProductSystem DWG bridge and fail closed rather
  than borrowing an obsolete Fadiorteam path or a temporary worktree copy.
- Keep the fork's `main` on the latest upstream history.
- Do not delete the legacy branch until a latest-upstream migration has passed
  equivalent CAD, Explorer, and robot-description acceptance checks.
