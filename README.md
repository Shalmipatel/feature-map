# Feature map skill

Drop [SKILL.md](SKILL.md) into `.cursor/skills/feature-map/` (or the Claude skills folder). The skill does two jobs: emit a pstack / poteto-mode verification map, then keep that map honest on every merge.

## What you get

- One markdown file per user-facing feature.
- Four H2s, in order: Sub-features, How to get to it (user POV), Driving it with the product harness, Gotchas.
- An index README with baseline preconditions.
- A merge-upkeep loop: index hygiene, source wave with citations, live pass, new-surface files, outcome (`clean` / `changed` / `blocked`).

The sample map is already written against Shalmi's public scanner:

[`poteto/features/`](poteto/features/)

Repo it verifies: https://github.com/Shalmipatel/mcp-approval-scanner

## Run it

**Generate**

1. Copy `SKILL.md` into the skills folder.
2. Open a real repo.
3. Drive the user surface. Record commands, stdout, and exit codes.
4. Emit `features/README.md` plus one file per feature, using the four-H2 skeleton in the skill.
5. Compare your first files to `poteto/features/` until the rhythm matches.

**Merge**

1. Open the PR diff and `features/README.md`.
2. Run the merge-upkeep loop in the skill.
3. End with `clean`, `changed`, or `blocked`.
