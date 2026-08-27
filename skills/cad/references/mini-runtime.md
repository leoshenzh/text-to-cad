# Mac mini CAD runtime

Read this reference when running, restoring, or diagnosing Leo's installed CAD environment.

## One execution host

All CAD execution runs on Mac mini:

- build123d modeling, STEP export, and geometry inspection;
- CAD Explorer preview and production build;
- QCAD DWG conversion.

Mac Studio does not run or hold a second CAD dependency environment. It may receive a request,
transfer files to `mac-mini`, invoke the mini runtime over SSH, and retrieve results. Synology
holds the ProductSystem quote helper and documentation; it is not a CAD execution host.

## Fixed mini paths

```text
CAD skill:        /Users/Leo/.claude/skills/text-to-cad/skills/cad
Claude entry:     /Users/Leo/.claude/skills/cad
Codex entry:      /Users/Leo/.codex/skills/cad
Python packages:  /Users/Leo/.local/share/fadior-cad/python-3.12
Python launcher:  scripts/mini-python
Node launcher:    scripts/mini-npm
Explorer deps:    scripts/explorer/node_modules
QCAD app:         /Applications/QCAD.app
```

Both agent entries point to the same installed mini skill. `mini-python` always invokes
`/opt/homebrew/opt/python@3.12/bin/python3.12` and supplies only the isolated CAD package path.
`mini-npm` supplies the Homebrew Node path explicitly because non-interactive SSH does not include
it by default.

## Runtime checks

Run from the CAD skill directory on mini:

```bash
scripts/mini-python -c 'import build123d, OCP; print("CAD_MODELING_READY")'
test -d scripts/explorer/node_modules
scripts/mini-npm --prefix scripts/explorer test
```

Generate and inspect a model with:

```bash
scripts/mini-python scripts/step /path/to/model.py -o /path/to/model.step
cd /path/to
/Users/Leo/.claude/skills/text-to-cad/skills/cad/scripts/mini-python \
  /Users/Leo/.claude/skills/text-to-cad/skills/cad/scripts/inspect \
  refs model.step --facts --planes --positioning
```

Start Explorer with an explicit scan root so it never scans the whole user directory:

```bash
scripts/mini-npm --prefix scripts/explorer run dev:ensure -- \
  --workspace-root /path/to/workspace \
  --file relative/path/to/model.step
```

For a production build, also set the workspace root explicitly:

```bash
EXPLORER_WORKSPACE_ROOT=/path/to/workspace EXPLORER_ROOT_DIR= \
  scripts/mini-npm --prefix scripts/explorer run build
```

## Restore dependencies on mini

Large downloads require Leo's approval before running these commands.

```bash
/opt/homebrew/opt/python@3.12/bin/python3.12 -m venv \
  /Users/Leo/.local/share/fadior-cad/python-3.12
/opt/homebrew/opt/python@3.12/bin/python3.12 -m pip install \
  --target /Users/Leo/.local/share/fadior-cad/python-3.12/lib/python3.12/site-packages \
  -r /Users/Leo/.claude/skills/text-to-cad/skills/cad/requirements.txt
scripts/mini-npm --prefix scripts/explorer ci
```

Do not install the same dependencies on Studio. Do not delete the Python or npm download caches
without Leo's approval.

## 2026-08-27 acceptance snapshot

- Modeling import: `CAD_MODELING_READY`; build123d 0.11.1, OCP 7.9.3.1.1, VTK 9.7.0.
- Representative STEP: a labeled plate with a through-hole generated successfully; inspection
  returned one solid, seven faces, and an 80 × 50 × 6 mm bounding box.
- Explorer: 147/147 tests passed; production build passed when given an explicit `/tmp` workspace;
  `dev:ensure` served the generated STEP and its GLB/topology catalog at `127.0.0.1:4178`.
- Dependency audit: zero known npm vulnerabilities after refreshing the locked versions.
- Installed size: Python environment 1.3 GB; Explorer `node_modules` 148 MB. Download caches were
  retained: pip 421 MB and npm 469 MB.
- DWG: ProductSystem helper live smoke produced a valid one-page PDF through QCAD 3.32.9 on mini.

This snapshot records a completed check, not a permanent guarantee. Re-run the runtime checks for
present-tense claims.
