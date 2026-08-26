---
name: feature-map
description: Use this when you need a verification feature map for a real repository, and again on every merge or PR that changes user-facing behavior. First run emits features/README.md plus one markdown file per feature (four H2s). Merge run keeps that map honest. User paths and observable proof only.
---

# Feature map (pstack / poteto-mode)

A feature map is how a fresh agent or engineer verifies a product from the outside. Lauren Tan (@poteto) at Cursor. The pstack `/create-verification-skill` shape.

One markdown file per user-facing feature. Four H2s, always in this order. User paths and observable proof only. Implementation stays in the source tree.

This skill does two jobs.

1. **Generate.** Interview the repo. Emit `features/README.md` plus one markdown file per user-facing feature.
2. **Merge upkeep.** On every merge, and on every PR that changes user-facing behavior, keep the map honest before the PR lands. Lauren Tan's pstack `/maintain-verification-skill` loop.

The sample that already holds this contract lives at [`poteto/features/`](poteto/features/). It maps the public repo [mcp-approval-scanner](https://github.com/Shalmipatel/mcp-approval-scanner). Open those files before you emit a new map.

## What you emit

```
features/README.md              index, baseline, conventions, feature list
features/<feature-slug>.md      one user-facing feature
```

Keep the map next to the repo you are verifying, or in a sibling `features/` directory you name in the index.

## Index README

Start with an H1 that names the product and calls the directory a verification map.

Then, in this order:

1. One short paragraph: read the index first, then drive the matching feature file.
2. `## Baseline preconditions`. Install, cwd, which fixtures or accounts to use, how to treat commands (literal), where proof artifacts live.
3. `## Driving conventions`. Fresh process vs shared state, the user surface (CLI, UI, API), the tokens that mean pass and fail, the rule for skipped checkpoints.
4. `## Feature entry contract`. H1, one behavior paragraph, then the four H2s in order. Copy the four names exactly.
5. `## Features`. A linked list. One line per file. The line is the user-visible behavior, the link is the slug file.

## Feature file contract

Every feature file uses this skeleton. Keep the H2 titles exact. Replace `<harness>` with the name of the user surface people actually drive (the CLI module, the web app, the API).

````markdown
# <User-visible behavior in a short title>

<One paragraph. What the user can do, and what they observe when it works.>

## Sub-features

- `<short-id>` <one line of behavior>
- `<short-id>` <one line of behavior>

## How to get to it (user POV)

- <every user entry point, as a command, URL, or click path>
- <the same feature reached from a second real entry point, if one exists>

## Driving it with <harness>

Preconditions:

- <install / cwd / fixture / account>
- <fresh process, or named shared state>
- <anything that must already be true>

- **<step name>.** Run `<exact command>`. <observable result>.
- **<step name>.** <what to read on screen>. <the token or row that proves it>.
- **Proof.** Exit code is `N`. Stdout contains `<token>`. Save stdout to `artifacts/<slug>/...`.

## Gotchas

- <a trap that wastes or invalidates this run>
- <a neighboring feature that looks like this proof and is a different file>
````

### Sub-features

Short kebab IDs. One line each. Each line is a slice of user-visible behavior you can point at in the drive section.

### How to get to it (user POV)

Entry points a person uses. Commands they type. Screens they open. Flags they pass. Keep source file paths out of this section.

### Driving it with the harness

Start with `Preconditions:`. Then pair every user action with an exact command and an observable result. Observables are exit codes, printed tokens (`SHIP: PASS`, `SHIP: FAIL`, `side=tools`), table rows, and files you save under `artifacts/`.

Name the harness after the product the user drives. In the sample that name is `mcp-approval-scanner`.

### Gotchas

Traps that invalidate a run: reused processes, the wrong `--calls` list, treating a neighboring feature as this proof, asserting a string the fixture never prints.

## Interview a repo and emit the first map

Work from the outside in.

1. **Drive the user surface.** Install what the README tells you to install. Run the documented commands. Record cwd, command, stdout, stderr, and exit code.
2. **List entry points.** CLI flags, URLs, buttons, fixture names. These become "How to get to it".
3. **Group by user-visible behavior.** One sentence per feature. A feature is something a person can succeed or fail at in one sitting. Split when the pass token changes (exit 0 vs exit 1, different fixture, different flag set).
4. **Name sub-features.** Short IDs for the slices inside that sitting (connect, pin, held, pass).
5. **Write the drive recipe.** Preconditions first. Then action, exact command, observable result. Save proof under `artifacts/<slug>/`.
6. **Write gotchas** from the first time you almost filed the wrong proof (short inspect vs full schedule, shared process, green pytest).
7. **Write the index last.** Baseline preconditions, driving conventions, the four-H2 contract, then the linked feature list.

Open [`poteto/features/`](poteto/features/) and copy the rhythm. Three features is enough for a first map of that scanner: pin holds, drift fails, short inspect stays under the gate.

## What counts as proof

- Exit code.
- A printed token you can quote (`SHIP: PASS`, `SHIP: FAIL`, `side=tools`, a named tool, a held prompts hash).
- A saved artifact path.

A green `pytest` run is a different signal. Drive the user surface and keep the pytest command in the product repo.

## Size

One feature file is one sitting at the user surface. If you need two pass tokens to tell the story (a passing one-call run and a failing 1,3,10 run), that pair is one feature when the point is the contrast. It is two features when each token is a result a user would file on its own. The sample splits them: `fail-on-drift.md` is the fail, `short-inspect-miss.md` is the contrast.

## Merge upkeep

Run this on every merge, and on every PR that changes user-facing behavior, before the PR lands. The map stays a live recipe. A stale file is a failed upkeep.

### What you may edit

Edit only the map: `features/` and this skill. Product source stays in its own review. If a mapped command fails because the app broke, report the command, the expected token, and the actual stdout. Leave the feature file pointing at that proof until the product is fixed. If the docs drifted (new flag, new token, new entry point) and the app still does the job, update the map in this same PR.

### Loop

1. **Index hygiene.** Open `features/README.md`. Every linked feature file exists. Every `features/*.md` except the index is linked. Drop dead links. Add a line for a file that is missing from the list.
2. **Source wave.** Read each feature file against the PR diff and the current source. Flag drift with citations: feature path plus heading, then the file and line (or diff hunk) that moved. Commands, tokens, entry points, and preconditions are the fields that can drift.
3. **Live pass.** Drive every mapped feature at least once on the real user surface. Fresh process. Capture stdout, stderr, exit code. Save proof under `artifacts/<slug>/` (the verifier writes that directory). A feature you cannot reach is `verified-unreachable`. Name the prerequisite that is missing (account, fixture, flag, environment).
4. **New surface.** A user-facing entry point in the diff with no feature file gets a new file in this same PR. Same four H2s. Add it to the index.
5. **Outcome.** End with exactly one of:
   - `clean`: live pass matched the map. No map edit.
   - `changed`: map files are in this PR. List them.
   - `blocked`: say what blocked (product regression, missing prerequisite, unreachable surface). The map still describes the last working proof.

### Source-wave citations

Write each drift as a pair.

- Map: `features/fail-on-drift.md` / Driving it with mcp-approval-scanner / "Exit code is `1`."
- Source: `src/mcp_approval_scanner/__main__.py:20` now returns 0 on drift.

A citation you cannot point at is a guess. Re-read the file.

### Live-pass proof

Reuse the generate-time proof rules. Exit code. A printed token. A saved artifact. `verified-unreachable` is a real result when the prerequisite is named. An untried feature is unfinished upkeep.

### When the app broke

Report it. Keep the feature file. The next product fix has to make that command print the token again. Rewriting the expected token to match a regression is how the map goes dead.

## Resume from a fresh context

**Generate.** Open `features/README.md` if it exists. If it is missing, interview the repo and emit the first map.

**Merge.** Open `features/README.md`, then run the merge-upkeep loop from index hygiene through outcome. Start from the PR diff when you have one.

1. Read baseline preconditions and driving conventions.
2. Pick the feature file that matches the behavior you are verifying.
3. Run that file's commands from a fresh process.
4. File proof under the artifact path it names.

The map is the recipe. The source tree stays closed until a command fails and you need to fix the product.

## Sample

Directory: [`poteto/features/`](poteto/features/)

| File | User-visible behavior |
| --- | --- |
| [README.md](poteto/features/README.md) | Index, baseline, conventions |
| [pin-stable-catalog.md](poteto/features/pin-stable-catalog.md) | Pin holds across 1, 3, and 10. Exit 0. |
| [fail-on-drift.md](poteto/features/fail-on-drift.md) | Mutating fixture rewrites `echo` at call 3. Exit 1. Prompts hold. |
| [short-inspect-miss.md](poteto/features/short-inspect-miss.md) | `--calls 1` on the same fixture still exits 0. |

Run the commands in `pin-stable-catalog.md` from a checkout of [mcp-approval-scanner](https://github.com/Shalmipatel/mcp-approval-scanner) to see a passing pin. Run `fail-on-drift.md` next to see the gate trip.
