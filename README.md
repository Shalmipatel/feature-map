# Feature map skill

Drop [SKILL.md](SKILL.md) into `.cursor/skills/feature-map/`.

The skill does two jobs.

1. **Generate** a project-local `.cursor/skills/verify-<app>/` skill (Launch, Doctor, Drive, Evidence, Cleanup, Helpers) plus a feature map inside that skill.
2. **Maintain** that map so it stays honest on every merge and every PR that changes user-facing behavior.

The next agent reads one feature file cold. The rest of the tree stays closed.

Shape is Lauren Tan (@poteto), pstack `/create-verification-skill` and `/maintain-verification-skill`:

- https://github.com/cursor/plugins/blob/main/pstack/skills/create-verification-skill/SKILL.md
- https://github.com/cursor/plugins/blob/main/pstack/skills/maintain-verification-skill/SKILL.md

## Sample

[`poteto/features/`](poteto/features/) is the feature-map slice for the public scanner.

Repo it verifies: https://github.com/Shalmipatel/mcp-approval-scanner

This skill repo: https://github.com/Shalmipatel/feature-map

## Run it

**Generate**

1. Copy `SKILL.md` into `.cursor/skills/feature-map/` of the agent that will do the work.
2. Open a real repo that already exists.
3. Interview the checkout (Surface, Run, Drive, Observe, Isolate). Ask the user only what you cannot observe.
4. Write `.cursor/skills/verify-<app>/SKILL.md` and seed `features/` with the four-H2 files. Start with the top 3 to 5.
5. Prove it: launch, doctor, drive one mapped feature, capture evidence, clean up, confirm the evidence still exists.
6. Compare the feature files to `poteto/features/` until the rhythm matches.

**Maintain**

1. Open the PR diff and `.cursor/skills/verify-<app>/features/README.md`.
2. Run the maintain loop in the skill (index hygiene, source wave, live pass, new surface).
3. End with `clean`, `changed`, or `blocked`.
4. A new user-facing change lands in the map in that same PR.
