# mcp-approval-scanner verification map

This directory is the maintained source for verifying the user-facing behavior of mcp-approval-scanner. Read the index before driving the CLI, then use the matching feature file as the recipe.

Shape follows Lauren Tan's pstack `/create-verification-skill` feature map. One file per feature. Four H2s in this order. User paths and observable proof only.

## Baseline preconditions

- Work from the repo root of `mcp-approval-scanner`.
- Install the package once with `python3 -m venv .venv && . .venv/bin/activate && pip install -e ".[dev]"`.
- Drive only the fixture servers in this repo (`fixtures/stable_server.py`, `fixtures/mutating_server.py`).
- Treat every command as literal. Keep quoted names and flags unchanged.
- Capture stdout, stderr, and exit code for every run. Proof artifacts live under `artifacts/` and survive cleanup.

## Driving conventions

- Start every recipe from a fresh process. The fixtures keep process-local call counts.
- The user surface is the CLI: `python -m mcp_approval_scanner --server <path> --calls <list>`.
- A passing pin prints `SHIP: PASS` and exits 0. A drifted pin prints `SHIP: FAIL` and exits 1.
- Do not report a skipped checkpoint as verified through a different `--calls` list.

## Feature entry contract

Each feature file starts with an H1 title and one paragraph describing the user-visible behavior. It then uses exactly four H2 sections in this order.

1. `Sub-features` lists short IDs with one line for each behavior.
2. `How to get to it (user POV)` lists every user entry point.
3. `Driving it with mcp-approval-scanner` starts with `Preconditions:` and pairs each user action with an exact command and observable result.
4. `Gotchas` lists traps that can waste or invalidate a verification run.

## Features

- [Pin a catalog that holds](./pin-stable-catalog.md) covers connect, pin, and exit 0 across checkpoints 1, 3, and 10.
- [Fail when the catalog rewrites](./fail-on-drift.md) covers the Deadbugz-shaped rewrite after three calls and the tools-vs-prompts split.
- [A short inspect stays under the gate](./short-inspect-miss.md) covers `--calls 1` exiting 0 against the mutating fixture.
