---
name: stress-test
description: Aggressive adversarial review of a coding or implementation plan — finds flaws, inconsistencies, poor fit with the existing project, unnecessary complexity or bloat, missed refactor opportunities, and tech debt. Asks clarifying questions inline via the AskUserQuestion tool when the right tradeoff depends on user intent, then returns a revised plan with a finding-disposition table and a what/why/how changelog. Use whenever the user asks to review, critique, poke holes in, stress-test, red-team, harden, or find flaws in a coding plan — even implicitly ("what's wrong with this plan", "am I missing anything", "review this before I build"). Skip for PR reviews, reviews of already-committed code, bug triage, or non-coding strategy plans.
---

# Stress-Test: Adversarial Coding Plan Review

Apply this skill when a coding plan has just been proposed and the user wants it attacked before implementation.

## Flow

### 1. Scope check
- Is there a plan in context? If not, ask the user to paste or point to it — do not invent one.
- Is the plan partially implemented? Read the relevant files and flag any divergence between plan and code.

### 2. Goal check (conditional)
If the plan's goal is ambiguous, restate it in one sentence and confirm. Otherwise skip — do not waste tokens restating a clear plan.

### 3. Read the code the plan touches
Read the files named in the plan, in the main context. Grep for key symbols only if the plan is vague about locations. Do not spawn an Explore subagent — the reviewer needs to quote specifics.

### 4. Critique along five axes
For each axis, either cite a concrete element of the plan (specific file, step, decision) with the problem and proposed fix, or explicitly pass with one sentence of reasoning. No vague complaints. No "looks good overall" summaries.

1. **Correctness & consistency** — logic errors, missed edge cases, race conditions, wrong assumptions about APIs, internal contradictions within the plan, and basic verifiability (how will the user know it worked?)
2. **Fit with the existing project** — conventions, patterns, CLAUDE.md rules, architectural constraints; does the plan match what's already there
3. **Simplicity & bloat** — *scope: what the plan adds.* Unnecessary files, deps, abstractions, config flags, feature-flag scaffolding, premature generality; is there a materially simpler approach. Cleanup of existing code belongs in axis 4.
4. **Refactor opportunities** — *scope: existing code the plan touches or reveals.* Adjacent cleanup that should happen in the same PR (in-scope) and adjacent cleanup that should *not* (out-of-scope, with reason). Be explicit about both lists. Additions the plan makes belong in axis 3.
5. **Tech debt** — debt paid, debt added, debt ignored

Per-axis concrete prompts live in [references/axes-checklist.md](references/axes-checklist.md). Load that file at the start of step 4.

### 5. Ask clarifying questions inline
Whenever a finding depends on user intent (which tradeoff to make, which style to match, whether to expand scope), ask via the **AskUserQuestion tool**. Rules:
- Use AskUserQuestion, never free-text prompts. The user wants clickable options.
- 2–4 suggested options per question, plus free-text fallback.
- Maximum 5 questions across the whole review.
- Only ask questions that would materially change a finding or the revised plan. No questions for the sake of questions.

### 6. Produce the revised plan
After the user answers, return a revised plan containing, in order:

1. **Updated plan** — full, standalone, ready to execute. Do not require the user to reconstruct it from diffs.

2. **Finding-disposition table** — every finding from step 4 appears here with status `accepted` / `mitigated` / `rejected` and a one-line reason. No silent drops.

   | Finding | Disposition | Reason |
   |---------|-------------|--------|
   | ...     | accepted    | ...    |

3. **Changelog** — what changed from the original plan, why, and how.

   | What | Why | How |
   |------|-----|-----|
   | ...  | ... | ... |

4. **Rejected alternatives** — only include if alternatives were actually considered during the review. Do not pad.

## Anti-patterns (refuse these)
- Rubber-stamp passes across all axes with no specifics
- Findings that don't cite a concrete plan element
- Silently dropping findings in the revised plan (use the disposition table)
- Restating a clear plan as a steelman when step 2 should have been skipped
- Free-text clarifying questions instead of AskUserQuestion
- Spawning Explore subagents — read in main context
- Attacking a strawman: if you misread the plan, fix the reading and re-critique

## Handling edge cases
- **No plan in context**: ask the user to paste or reference it. Do not fabricate.
- **Iteration**: the skill is re-invokable. If the user says "review the revision" or re-runs the skill on the updated plan, rerun the full flow.
- **Very large plan**: critique the highest-leverage axes first and tell the user which axes you deferred and why.
