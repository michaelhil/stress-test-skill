# stress-test

Aggressive adversarial review skill for Claude Code. Stress-tests a coding plan across five axes (correctness, project fit, simplicity/bloat, refactor opportunities, tech debt), asks clarifying questions inline via `AskUserQuestion`, and returns a revised plan with a finding-disposition table and a what/why/how changelog.

## Install

Symlink the skill directory into your Claude skills folder:

```bash
ln -s "$(pwd)/skill" ~/.claude/skills/stress-test
```

Then invoke by asking Claude to stress-test, adversarially review, poke holes in, red-team, or harden a plan.

## Layout

- `skill/SKILL.md` — entry point loaded by Claude when the skill triggers
- `skill/references/axes-checklist.md` — per-axis concrete prompts, loaded at step 4
- `tests/triggers.md` — should / shouldn't trigger evaluation list; eyeball after editing the description
