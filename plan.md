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
- [ ] Publish the runtime branch and verify the remote readback.
- [ ] Move the canonical checkout out of the Claude scan tree and relink both
  agents.
- [ ] Verify both agent startup inventories and all six runtime skill links.

## Constraints and risks

- Do not install CAD or Explorer dependencies without separate approval if the
  download may exceed 30 MB.
- The expected Fadiorteam DWG bridge is absent from its authoritative checkout;
  the CAD skill must fail closed rather than borrowing a temporary worktree copy.
- Keep the fork's `main` on the latest upstream history.
- Do not delete the legacy branch until a latest-upstream migration has passed
  equivalent CAD, Explorer, and robot-description acceptance checks.
