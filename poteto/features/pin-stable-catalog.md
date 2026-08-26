# Pin a catalog that holds

Pin a catalog that holds lets a user connect to a stdio MCP server, hash the tool and prompt catalogs, call the tools, and confirm the pin is unchanged at every checkpoint.

## Sub-features

- `pin-connect` speaks MCP 2026-07-28 on stdio and snapshots `tools/list` plus every `prompts/get`.
- `pin-hash` prints `pin.tools`, `pin.prompts`, and `pin.combo`.
- `pin-held` reprints the same `pin.tools` after 1, 3, and 10 calls.
- `pin-pass` exits 0 and prints `SHIP: PASS`.

## How to get to it (user POV)

- From the repo root, run `python -m mcp_approval_scanner --server fixtures/stable_server.py --calls 1,3,10`.
- Point `--server` at any other stdio MCP script whose catalog is expected to hold.

## Driving it with mcp-approval-scanner

Preconditions:

- The package is installed editable with the `dev` extra.
- `fixtures/stable_server.py` is the server under test.
- A fresh process is started for this recipe.

- **Connect and pin.** Run `python -m mcp_approval_scanner --server fixtures/stable_server.py --calls 1,3,10`. Stdout names the server path and prints three SHA-256 pins.
- **Read the table.** The report has rows for after 1, 3, and 10. Each row shows `held` and `OK`.
- **Confirm the hash.** The `tools_hash` on every row matches the printed `pin.tools`.
- **Proof.** Exit code is `0`. Stdout ends with `SHIP: PASS`. Save stdout to `artifacts/pin-stable/report.txt`.

## Gotchas

- Reusing a long-lived mutating fixture process will poison this recipe. Start a new process.
- Key order on the wire cannot change the pin. If a hash moves with no description change, the canonicalize step is broken.
- A green pytest run is not this proof. Drive the CLI.
