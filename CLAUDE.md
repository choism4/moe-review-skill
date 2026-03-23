# MoE Review Skill

Mixture-of-Experts parallel review for Claude Code. Assembles 5 domain-specific virtual experts to review any deliverable in parallel, auto-applies fixes, and loops until all experts approve.

## Project Structure

```
SKILL.md          — Skill definition (the actual skill that Claude Code loads and executes)
README.md         — User-facing documentation, install instructions, usage examples
CLAUDE.md         — This file. Project context for contributors.
LICENSE           — MIT
.gitignore        — Standard ignores
```

This is a **markdown-only skill**. There are no source code files, no build steps, no tests, and no dependencies.

## How It Works

- SKILL.md defines the full review protocol: analyze, assemble experts, parallel review, synthesize, fix, re-review, report.
- The coordinator dispatches 5 expert reviewers via the **Agent tool** simultaneously in a single function_calls block.
- Each expert has a specific persona (name, background, company, specialty) to produce deep, non-generic reviews.
- The review-fix-re-review loop runs automatically up to 3 rounds.
- Works on any deliverable type: code diffs, papers, reports, design docs, proposals, etc.

## Build / Test

None. This project contains only markdown files. There is nothing to build or test.

## Conventions

- All 5 expert Agent calls MUST be dispatched in a single function_calls block (parallel, never sequential).
- Expert personas must be specific: name, former company, years of experience, specialty.
- The 5 experts must cover 5 different axes — diversity over depth.
- Auto-fix is conservative: CRITICAL/MAJOR are fixed immediately, design judgments are escalated to the user.
- Maximum 3 review rounds. Report and stop if unresolved.
- Final output uses table format for scannability.

## Do NOT

- Do not add code files. This is a markdown-only skill.
- Do not rename the skill. It must remain `moe-review` in SKILL.md frontmatter.
- Do not run expert agents sequentially. Parallel execution is the skill's core value.
- Do not use fewer than 5 experts or duplicate expertise axes.
- Do not auto-fix issues that require design judgment — escalate those to the user.
- Do not exceed 3 review rounds. Report remaining issues and stop.
