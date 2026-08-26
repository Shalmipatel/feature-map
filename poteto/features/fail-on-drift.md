# Fail when the catalog rewrites

Fail when the catalog rewrites lets a user pin a server, use it past the Deadbugz threshold, and get an exit 1 that names the tool whose description moved while prompts held.

## Sub-features

- `drift-ok-early` stays green at checkpoint 1 against the mutating fixture.
- `drift-trip` exits 1 at checkpoint 3.
- `drift-name` names `echo` and prints both tool hashes.
- `drift-side` reports `side=tools` and `prompts (held)`.

## How to get to it (user POV)

- From the repo root, run `python -m mcp_approval_scanner --server fixtures/mutating_server.py --calls 1,3,10`.
- Point `--server` at a live MCP server after it has been approved, using the same `--calls 1,3,10` schedule.

## Driving it with mcp-approval-scanner

Preconditions:

- The package is installed editable with the `dev` extra.
- `fixtures/mutating_server.py` is the server under test. Its `THRESHOLD` is 3.
- A fresh process is started for this recipe.

- **Connect and pin.** Run `python -m mcp_approval_scanner --server fixtures/mutating_server.py --calls 1,3,10`. The first pin matches the stable fixture pin.
- **Read checkpoint 1.** The after-1 row shows `held` and `OK`.
- **Read checkpoint 3.** The after-3 row shows `changed` and `DRIFT`. `side=tools`. The diff names `echo` and the suffix `[DRIFT after 3 tools/call]`.
- **Read prompts.** The prompts hash is printed as held.
- **Proof.** Exit code is `1`. Stdout contains `SHIP: FAIL`. Save stdout to `artifacts/fail-on-drift/report.txt`.

## Gotchas

- `--calls 1` against this fixture exits 0. That is a different feature. Do not treat it as this proof.
- The fixture does not carry a credential-seeking payload. Assert the labeled description rewrite, not a secret-stealing string.
- Stopping at the first inspect is the miss this feature exists to catch.
