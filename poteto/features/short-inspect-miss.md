# A short inspect stays under the gate

A short inspect stays under the gate lets a user see why a one-call smoke test is insufficient. The mutating fixture still looks approved after a single `tools/call`.

## Sub-features

- `inspect-one` runs `--calls 1` against the mutating fixture.
- `inspect-pass` exits 0 and prints `SHIP: PASS`.
- `inspect-contrast` is only complete when the same server fails at `--calls 1,3,10`.

## How to get to it (user POV)

- From the repo root, run `python -m mcp_approval_scanner --server fixtures/mutating_server.py --calls 1`.
- Compare it immediately with `--calls 1,3,10` on the same fixture.

## Driving it with mcp-approval-scanner

Preconditions:

- The package is installed editable with the `dev` extra.
- `fixtures/mutating_server.py` is the server under test.
- Two fresh processes, one per command.

- **Run the short inspect.** Run `python -m mcp_approval_scanner --server fixtures/mutating_server.py --calls 1`. Exit code is `0`. Stdout ends with `SHIP: PASS`.
- **Run the full schedule.** In a new process, run `python -m mcp_approval_scanner --server fixtures/mutating_server.py --calls 1,3,10`. Exit code is `1`. Stdout ends with `SHIP: FAIL` at checkpoint 3.
- **Proof.** Save both reports under `artifacts/short-inspect/one.txt` and `artifacts/short-inspect/full.txt`. The pair is the proof. The passing one-call run alone is not.

## Gotchas

- The two commands must be separate processes. A shared process has already counted the first calls.
- Do not file the passing `--calls 1` run as a stable-catalog proof.
