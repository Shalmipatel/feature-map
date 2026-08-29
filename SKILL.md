---
name: feature-map
description: Use this when a repo already exists and you need a project-local verify skill + feature map so the next agent can read one feature file cold instead of re-interviewing the tree; and again on every merge/PR that changes user-facing behavior.
---

# Feature map

Lauren Tan (@poteto). pstack `/create-verification-skill` and `/maintain-verification-skill`. This skill generates a project-local verification skill. The next agent reads that skill cold, mid-task, having never seen the app.

The feature map lives inside that generated skill. One feature file opens for the change under test. The rest of the tree stays closed. That is the token cut. A new user-facing change in a PR gets a map update in that same PR.

Official shape:

- https://github.com/cursor/plugins/blob/main/pstack/skills/create-verification-skill/SKILL.md
- https://github.com/cursor/plugins/blob/main/pstack/skills/maintain-verification-skill/SKILL.md

The sample map lives at [`poteto/features/`](poteto/features/). It maps the public repo [mcp-approval-scanner](https://github.com/Shalmipatel/mcp-approval-scanner). Open those files before you emit a new map.

This skill does two jobs.

1. **Generate.** Interview the repo. Write `.cursor/skills/verify-<app>/` (skill body plus a seeded feature map). Prove it once. Hand it to the next agent.
2. **Maintain.** On every merge, and on every PR that changes user-facing behavior, keep that map honest before the PR lands.

## What you emit

```
.cursor/skills/verify-<app>/SKILL.md
.cursor/skills/verify-<app>/features/README.md
.cursor/skills/verify-<app>/features/<feature-slug>.md
.cursor/skills/verify-<app>/helpers/     optional; executable; cited from the skill body
```

`<app>` is a short kebab name taken from the repo. For the sample that name is `mcp-approval-scanner`.

A `features/` folder at the repo root is the wrong target. The map is an input to the next agent. It belongs inside the skill that agent will load.

## Generate (pstack /create-verification-skill)

### 1. Interview the repo, not the user

Read the checkout. Derive five facts. Ask the user only for a fact you cannot observe.

| Fact | You are answering |
| --- | --- |
| Surface | The user-facing thing a person drives (CLI module, URL, desktop window, API). |
| Run | Exact start or invoke command. Ready signal. How a short-lived CLI differs from a long-lived server. |
| Drive | The harness: real selectors, flags, or commands in THIS repo. |
| Observe | Tokens, rows, files, screens, exit codes that prove a path. |
| Isolate | What makes a session independent (fresh process, fixture, account, cwd). |

If the checkout does not start, fix it or report the failure. Do not generate a skill against a dead tree.

### 2. Write the verify skill

Write `.cursor/skills/verify-<app>/SKILL.md` with YAML frontmatter:

- `name`: `verify-<app>`
- `description`: names the app, names the surface, and says when to reach for this skill (a mid-task agent about to drive this product)

Then six sections, grounded in the interview. No placeholders. No `TODO`. Every command is one you ran or read from this repo.

**Launch.** Exact start command. Ready signal you can quote. Teardown for a long-lived process. For a short-lived CLI: build once, isolated session per drive.

**Doctor.** Read-only. Is this instance worth driving? Cwd, binary, fixture, health token. A sick instance fails doctor. You do not drive it.

**Drive.** Harness recipe. Real selectors and commands from this repo. Point at `features/` for the per-feature recipes. The next agent opens one feature file and follows that file.

**Evidence.** What to capture, the directory it lands in, and the proof standard:

- A real user path.
- An action plus the resulting state.
- Side effects you can name (exit code, written file, printed token).
- Mocks only at a real production boundary.

**Cleanup.** Kill what you started. Record the pid at Launch. Never kill by process name. Evidence survives cleanup.

**Helpers.** If you add a script, it is executable, and the skill body shows the invocation. A helper with no call site is dead weight.

### 3. Seed the feature map

Write `.cursor/skills/verify-<app>/features/README.md` plus one file per user-facing feature. Start with the top 3 to 5. User paths and observable proof only. Implementation stays in the source tree.

Index first:

1. H1 that names the product and calls the directory a verification map.
2. One short paragraph: read the index, then open the matching feature file.
3. `## Baseline preconditions`. Install, cwd, fixtures or accounts, literal commands, where proof artifacts live.
4. `## Driving conventions`. Fresh process vs shared state, the user surface, pass and fail tokens, the rule for a skipped checkpoint.
5. `## Feature entry contract`. H1, one behavior paragraph, then the four H2s in order.
6. `## Features`. A linked list. One line per file. The line is the user-visible behavior.

Every feature file uses this skeleton. Keep the H2 titles exact. Replace `<harness>` with the surface people drive.

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

**Sub-features.** Short kebab IDs. One line each. Each line is a slice of user-visible behavior you can point at in the drive section.

**How to get to it (user POV).** Entry points a person uses. Commands they type. Screens they open. Flags they pass. Keep source file paths out of this section.

**Driving it with the harness.** Start with `Preconditions:`. Then pair every user action with an exact command and an observable result. Observables are exit codes, printed tokens, table rows, screens, and files you save under `artifacts/`. Name the harness after the product the user drives. In the sample that name is `mcp-approval-scanner`.

**Gotchas.** Traps that invalidate a run: reused processes, the wrong flag set, treating a neighboring feature as this proof, asserting a string the fixture never prints.

Open [`poteto/features/`](poteto/features/) and copy the rhythm. Three files are enough for a first map of that scanner: pin holds, drift fails, short inspect stays under the gate.

### 4. Prove the generated skill

Before you hand the skill over, execute it.

1. Launch.
2. Doctor.
3. Drive one mapped feature.
4. Capture evidence where the skill says it goes.
5. Cleanup.
6. Confirm the evidence still exists.

A skill that was never executed is a draft. Say so, or finish the prove pass.

### 5. Point at the maintain loop

The generated skill is live the moment you prove it. From here, every user-facing change updates the map in the same change. The maintain job below is that loop.

## Feature file size

One feature file is one sitting at the user surface. Two pass tokens can share a file when the point is the contrast. They split when each token is a result a user would file on its own. The sample splits them: `fail-on-drift.md` is the fail, `short-inspect-miss.md` is the contrast.

## What counts as proof

- Exit code.
- A printed token you can quote (`SHIP: PASS`, `SHIP: FAIL`, `side=tools`, a named tool, a held prompts hash).
- A saved artifact path.
- A screen that shows the action and the resulting state.

A green `pytest` run is a different signal. Drive the user surface and keep the pytest command in the product repo.

## Maintain (pstack /maintain-verification-skill)

The feature map rots the moment the app changes. The unit of rigor is the feature.

Edit only the verify skill directory: `.cursor/skills/verify-<app>/`. Product source stays in its own review.

Never paper over a product regression by rewriting the map. If the app broke, report the command, the expected token, and the actual output. Leave the feature file pointing at that proof until the product is fixed. If the docs drifted (new flag, new token, new entry point) and the app still does the job, fix the map in this same PR.

Any new change an agent makes that is user-facing is added to the feature map in that same change.

### Loop

1. **Index hygiene.** Open `features/README.md` inside the verify skill. Every linked feature file exists. Every `features/*.md` except the index is linked. Drop dead links. Add a line for a file that is missing from the list.
2. **Source wave.** Read each feature file against the PR diff and the current source. Flag drift with citations: feature path plus heading, then the file and line (or diff hunk) that moved. Commands, tokens, entry points, and preconditions are the fields that can drift.
3. **Live pass.** Drive every mapped feature at least once on the real user surface. Fresh process. Capture stdout, stderr, exit code, or the matching screen proof. Save proof under the path the feature file names. A feature you cannot reach is `verified-unreachable`. Name the prerequisite that is missing (account, fixture, flag, environment).
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

## How the next agent uses the generated skill

The next agent has never seen this app. It loads `verify-<app>` mid-task.

1. Read Launch and Doctor. Start the instance, or report that doctor failed.
2. Open exactly one feature file, the one that matches the change under test.
3. Drive that file. Leave the other feature files closed. Leave the product tree closed unless a command fails and the product needs a fix.
4. File evidence. Cleanup. Evidence stays.
5. If this agent just added user-facing behavior, add or edit the matching feature file in the same change, then run the maintain loop.

That closed tree is the point. Re-interviewing the repo on every task is the cost this skill exists to cut.

## Resume

**Generate.** If `.cursor/skills/verify-<app>/SKILL.md` is missing, interview the repo and emit the skill plus the first map. Then prove it.

**Maintain.** Open the verify skill's `features/README.md`. Run the maintain loop from index hygiene through outcome. Start from the PR diff when you have one.

## Sample

Directory: [`poteto/features/`](poteto/features/)

This is the feature-map slice for mcp-approval-scanner. A generate pass for that repo wraps these files in `.cursor/skills/verify-mcp-approval-scanner/` with Launch, Doctor, Drive, Evidence, Cleanup, and Helpers derived from that CLI.

| File | User-visible behavior |
| --- | --- |
| [README.md](poteto/features/README.md) | Index, baseline, conventions |
| [pin-stable-catalog.md](poteto/features/pin-stable-catalog.md) | Pin holds across 1, 3, and 10. Exit 0. |
| [fail-on-drift.md](poteto/features/fail-on-drift.md) | Mutating fixture rewrites `echo` at call 3. Exit 1. Prompts hold. |
| [short-inspect-miss.md](poteto/features/short-inspect-miss.md) | `--calls 1` on the same fixture still exits 0. |

Run the commands in `pin-stable-catalog.md` from a checkout of [mcp-approval-scanner](https://github.com/Shalmipatel/mcp-approval-scanner) to see a passing pin. Run `fail-on-drift.md` next to see the gate trip.
