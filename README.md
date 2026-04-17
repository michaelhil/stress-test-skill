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

## Using this with other coding agents (Copilot, ChatGPT, Cursor, etc.)

This skill targets Claude Code. The Anthropic-specific pieces — the `SKILL.md` frontmatter, progressive disclosure of reference files, and the `AskUserQuestion` tool for clickable clarifying questions — will not work as-is on other systems. The *content* (the 5 axes, the flow, the finding-disposition table, the what/why/how changelog) is just prompt engineering and ports fine.

The easiest way to adapt it is to ask your own coding agent to do the porting. Paste this prompt into your agent, along with the contents of `skill/SKILL.md` and `skill/references/axes-checklist.md` from this repo:

> I want to adapt the attached Claude Code skill to work with **[your system: Copilot / ChatGPT custom instructions / Cursor rules / Aider / Continue / etc.]**. The skill performs adversarial review of a coding plan across five axes.
>
> Please:
> 1. Identify the parts that are Claude-specific (SKILL.md frontmatter, `AskUserQuestion` tool calls, reference-file loading, progressive disclosure) and rewrite or remove them for my system.
> 2. Merge the SKILL.md body and the axes-checklist content into a single prompt or rules file in my system's native format (e.g. `.github/prompts/stress-test.prompt.md` for Copilot, a custom GPT instruction, `.cursorrules` entry, `CONVENTIONS.md` entry for Aider).
> 3. Replace `AskUserQuestion` with plain-text clarifying questions the model asks inline during the review.
> 4. Preserve the 5 axes, the finding-disposition table, and the what/why/how changelog — these are the core of the skill.
> 5. Keep the trigger phrases in mind so I can invoke it naturally in my system (on systems without auto-triggering skills, I will invoke it manually by prefix or slash-command).
>
> Output the adapted file, ready for me to drop into my project, plus a one-paragraph note on how to invoke it in my system.

Review the output before using it — the adaptation is best-effort, not verified.
