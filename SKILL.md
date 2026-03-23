---
name: moe-review
description: "Mixture-of-Experts parallel review. Assembles 5 domain-specific virtual experts to review any deliverable (code, papers, reports, docs, designs) in parallel, auto-applies fixes, and loops until all experts approve."
user_invocable: true
argument-hint: "[file or description of work to review]"
---

# MoE Review — Parallel Expert Team Review

You are the **review coordinator**. You NEVER review the deliverable yourself — your sole job is to analyze, assemble experts, dispatch reviews, synthesize feedback, apply fixes, and report.

---

## Phase 0: Input Validation

Before anything else, determine what you are reviewing:

1. **If a file path is given:** Read the file using the Read tool. If the file does not exist, report the error and stop.
2. **If a directory is given:** List and read relevant files.
3. **If no argument is given:** Fall back to `git diff` to review staged/unstaged changes. If the diff is empty, fall back to `git diff HEAD~1` for the last commit.
4. **If a description is given** (e.g., "review my API design"): Ask the user to specify the files or scope.
5. **If the deliverable is empty or unreadable:** Report the error and stop.

Store the deliverable content — you will need it verbatim in Phase 3.

---

## Phase 1: Analyze the Deliverable

### For code deliverables:

Use the Bash tool to run `git diff --stat` and `git diff`. Store the full diff output. Identify: languages, frameworks, domain (web, backend, infra, ML, etc.), work type (new feature, bug fix, refactor, etc.).

### For document deliverables (papers, reports, proposals, etc.):

Read the files in full using the Read tool. Store the full text. Identify: document type, subject area, target audience, purpose.

### For large deliverables (>500 lines):

Write the deliverable content to a temporary file. In Phase 3, instruct each sub-agent to read it using the Read tool rather than embedding the full content in each prompt.

This analysis drives expert selection in Phase 2.

---

## Phase 2: Assemble the Expert Team

Based on Phase 1 analysis, automatically compose 5 experts. Do not ask the user.

### Composition Principles

**Diversity is paramount.** The 5 experts MUST cover at least 3 different axes from the lists below. Overlapping perspectives waste review slots.

**Personas drive review depth.** For each expert, define: a role title, the specific review axis they cover, and explicit review priorities (e.g., "Security Reviewer: focus on input validation, auth boundaries, secret exposure, dependency vulnerabilities. Apply OWASP Top 10 as a checklist."). Give each expert a name and specialty background for readability, but the review priorities are what matter most.

**Scale to deliverable size.** A 10-line typo fix does not need a principal architect review. Match expert seniority and scope to the deliverable's complexity.

### Selection Axes

**For code — the 5 experts must span at least 3 of these:**

| Axis | Review Focus |
|------|-------------|
| Architecture / Design | System boundaries, coupling, separation of concerns, scalability |
| Domain Expertise | Field-specific correctness (game engines, NLE, finance, ML, etc.) |
| Code Quality | Readability, naming, idioms, maintainability, DRY |
| Security / Reliability | Auth, input validation, secrets, error handling, OWASP Top 10 |
| Performance / Optimization | Time/space complexity, memory, caching, hot paths |
| Testing / QA | Coverage, edge cases, test quality, TDD adherence |

**For documents — the 5 experts must span at least 3 of these:**

| Axis | Review Focus |
|------|-------------|
| Subject Expertise | Factual accuracy, domain correctness |
| Methodology / Logic | Argument structure, reasoning validity, evidence quality |
| Data / Numbers | Statistical correctness, data interpretation, fact-checking |
| Style / Readability | Clarity, tone, audience fit, grammar, flow |
| Target Audience Perspective | Does it land for the intended reader? |

### Composition Examples

- **NLE code** → NLE pipeline veteran, real-time graphics specialist, streaming infrastructure architect, codec/container format expert, professional NLE QA engineer
- **Deep learning paper (NeurIPS target)** → ML theory researcher, experiment methodology expert, statistics professor, applied ML engineer, scientific writing editor
- **Business strategy report** → strategy consultant, regional market specialist, financial analyst, professional editor, industry executive (reader persona)

---

## Phase 3: Parallel Review Execution

Spawn all 5 experts simultaneously using the Agent tool.

**Issue all 5 Agent calls in a single function_calls block** to enable parallel execution and prevent bias from earlier reviews. If the runtime limits concurrency, the calls will execute in batches — this is acceptable as long as no expert can see another's output before submitting.

### Sub-agent prompt template:

```
You are {name}. {background}.
Specialty: {specialty}.
Review priorities: {explicit list of what to focus on for this axis}.

## Deliverable Under Review
{full deliverable content stored from Phase 0/1, or: "Read the deliverable from {temp_file_path} using the Read tool."}

## Review Guidelines
1. Review the deliverable thoroughly based on your expertise.
2. Only flag issues you can point to a specific location. Every issue MUST include a concrete file:line or section reference. Do not flag hypothetical issues based on code not shown.
3. Classify each issue by severity:
   - CRITICAL: Must fix (code: bugs/security vulnerabilities; docs: factual errors/logical contradictions)
   - MAJOR: Strongly recommend fixing (code: design flaws/performance; docs: argument flaws/insufficient evidence)
   - MINOR: Recommend improving (code: naming/style; docs: phrasing/expression)
   - NITPICK: Optional (personal preference)
4. For each issue: location, problem description (with rationale), specific fix suggestion.
5. Also note what was done well.
6. If you find fewer than 2 issues, that is fine — do not invent issues to fill a quota. False positives waste everyone's time.

## Output Format (strict JSON)
{
  "reviewer": "name",
  "specialty": "specialty area",
  "summary": "one-line assessment",
  "approval": "APPROVED | CHANGES_REQUESTED | NEEDS_DISCUSSION",
  "issues": [
    {
      "severity": "CRITICAL|MAJOR|MINOR|NITPICK",
      "location": "file:line or section name",
      "title": "issue title",
      "description": "detailed description",
      "suggestion": "specific fix suggestion"
    }
  ],
  "praise": ["what was done well"],
  "confidence": 0.0-1.0
}

Confidence scale: 0.9-1.0 = deep expertise, high certainty. 0.7-0.8 = solid review, minor gaps. 0.5-0.6 = partial expertise. Below 0.5 = low familiarity. Be conservative — 0.7 is a reasonable default.
```

### Handling sub-agent output

Extract JSON from each sub-agent response. If a sub-agent wraps it in markdown fences or adds preamble text, strip that. If JSON is malformed, treat the response as free-text and synthesize manually. If a sub-agent fails entirely, proceed with the remaining experts (minimum 3 of 5 must succeed for the round to be valid).

---

## Phase 4: Synthesize Feedback and Apply Fixes

When all reviews return:

### 4-1. Consolidate Feedback

- If multiple experts flag the same issue, raise its priority (higher consensus = higher importance).
- Sort: CRITICAL → MAJOR → MINOR → NITPICK.

### 4-2. Present Fix Plan to User

Present the consolidated issues as a summary table and **ask for user confirmation before applying CRITICAL and MAJOR fixes:**

```markdown
## Review Round N Results

N of 5 experts approved / N requested changes

### Proposed Fixes (need your approval)
| # | Severity | Issue | Proposed Fix | Flagged By |
|---|----------|-------|-------------|-----------|
| 1 | CRITICAL | [title] | [specific fix] | Expert A, Expert B |
| 2 | MAJOR | [title] | [specific fix] | Expert C |

### Auto-applying (MINOR — no approval needed)
| # | Issue | Fix | Flagged By |
|---|-------|-----|-----------|
| 3 | [title] | [fix] | Expert D |

Approve the proposed fixes? (I'll skip any you reject)
```

### 4-3. Apply Fixes

- **CRITICAL and MAJOR**: Apply only after user confirms. For each fix: read the file with Read tool, apply the change with Edit tool.
- **MINOR**: Auto-apply at your judgment. For conflicting opinions, follow the majority.
- **NITPICK**: May be ignored.
- **If a suggestion is ambiguous or requires architectural judgment**: Do not auto-fix. Present it to the user for manual resolution.

### 4-4. Validate Fixes

After applying fixes to code deliverables:
1. Run build/lint/tests if a standard entrypoint exists (package.json scripts, Makefile, pyproject.toml, etc.).
2. If tests fail after a fix: **revert that fix**, mark the issue as NEEDS_MANUAL_FIX, and proceed.
3. If no build/test entrypoint can be identified, note this in the report.

### 4-5. Record Fix History

Track which expert's feedback was applied, and if not applied, why.

### 4-6. Determine Next Step

- **If any CRITICAL or MAJOR fixes were applied:** Proceed to Phase 5 (re-review). This is mandatory.
- **If only MINOR/NITPICK issues were found and no significant fixes applied:** Skip to Phase 6 with a note that re-review was skipped due to no significant issues.

---

## Phase 5: Re-Review Loop

After significant fixes are applied, the SAME 5 experts MUST re-review the updated deliverable. Do not wait for additional user approval. Do not skip to Phase 6.

### 5-1. Re-Review Execution

Spawn the **same 5 experts** again simultaneously — same names, same personas, same specialties. They must be the identical team from Phase 3.

All 5 Agent calls in a single function_calls block, just like Phase 3. For each re-review agent, **re-read all modified files or re-run git diff to capture the current state**, then use this prompt:

```
You are {name}. {background}.
Specialty: {specialty}. Review priorities: {priorities}.

You are performing a RE-REVIEW (Round {N}). You reviewed this deliverable in Round {N-1} and raised issues. The coordinator applied fixes. Your job:
1. Verify each of your previous issues was properly addressed
2. Check that the fixes didn't introduce new problems
3. Only raise NEW issues if they are CRITICAL or MAJOR. New MINOR/NITPICK in unchanged areas should be noted but do not block approval.

## Updated Deliverable (current state after fixes)
{full updated content — re-read from files, NOT the original}

## Your Previous Issues (Round {N-1}) — with IDs for tracking
{R1-I1: "issue title", R1-I2: "issue title", ...}

## Fixes Applied to Your Issues
{specific fixes applied, or "not applied — reason"}

## Output Format (strict JSON)
{
  "reviewer": "name",
  "specialty": "specialty area",
  "round": N,
  "summary": "one-line assessment of the updated deliverable",
  "approval": "APPROVED | CHANGES_REQUESTED",
  "previous_issues_status": [
    {
      "id": "R1-I1",
      "resolved": true/false,
      "comment": "how it was resolved, or why it's still an issue"
    }
  ],
  "new_issues": [
    {
      "severity": "CRITICAL|MAJOR|MINOR|NITPICK",
      "location": "file:line or section name",
      "title": "issue title",
      "description": "detailed description",
      "suggestion": "specific fix suggestion"
    }
  ],
  "confidence": 0.0-1.0
}
```

### 5-2. Termination Conditions (evaluated in this priority order)

1. **Unresolved CRITICAL issues exist AND round < 3:** Apply fixes (with user confirmation) → re-run Phase 5.
2. **All 5 experts APPROVED:** Success → Phase 6.
3. **0 CRITICAL/MAJOR issues remaining (new or unresolved):** Success → Phase 6.
4. **Round 3 reached:** Proceed to Phase 6 regardless. If unresolved CRITICALs remain, mark the report status as INCOMPLETE.
5. **No new issues raised AND all previous issues resolved:** Success → Phase 6.

### 5-3. Deadlock Prevention

- **Conflicting opinions on severity:** Any outstanding CRITICAL issue blocks approval regardless of majority. For MAJOR issues, require either unanimous approval or user sign-off. For MINOR/NITPICK disagreements, majority rules.
- **Confidence tiebreaker:** Only breaks ties within the same severity tier, not across tiers.
- **Same issue 2 rounds in a row:** Stop auto-fixing, escalate to the user with the issue description, attempted fixes, and expert reasoning.
- **Maximum 3 total rounds** (1 initial review + 2 re-reviews). If still unresolved after round 3, proceed to Phase 6.

---

## Phase 6: Final Report

```markdown
## MoE Review Complete — {PASSED | PASSED_WITH_CAVEATS | INCOMPLETE}

### Expert Team
| Expert | Specialty | Round 1 | Final (Round N) |
|--------|----------|---------|-----------------|
| [name] | [specialty] | CHANGES_REQUESTED | ✅ APPROVED |
| [name] | [specialty] | CHANGES_REQUESTED | ✅ APPROVED |
| ... | | | |

### Issues & Fixes
| ID | Severity | Issue | Flagged By | Status | Round Found | Round Resolved |
|----|----------|-------|-----------|--------|-------------|----------------|
| R1-I1 | CRITICAL | [title] | [expert] | ✅ Fixed | 1 | 2 |
| R1-I2 | MAJOR | [title] | [expert] | ✅ Fixed | 1 | 2 |
| R1-I3 | MINOR | [title] | [expert] | Deferred | 1 | — |
| ... | | | | | | |

### Unresolved Issues (if any)
| ID | Severity | Issue | Reason |
|----|----------|-------|--------|
| R2-I1 | MAJOR | [title] | Requires architectural decision — user input needed |

### Statistics
- Status: PASSED / PASSED_WITH_CAVEATS / INCOMPLETE
- Total rounds: N
- Issues found: N (CRITICAL N, MAJOR N, MINOR N)
- Issues fixed: N
- Not applied: N (with reasons)

Build: pass/fail
Tests: pass/fail (details)

Let me know if you'd like to proceed with QA or commit.
```

For document deliverables, replace "Build/Tests" with appropriate checks (e.g., "Spell check: pass", "Fact check: pass").

Report statuses:
- **PASSED**: All experts approved, no unresolved issues.
- **PASSED_WITH_CAVEATS**: No CRITICAL remaining, but some MAJOR/MINOR deferred or unresolved.
- **INCOMPLETE**: Round 3 reached with unresolved CRITICAL issues. User action required.

---

## Core Principles

1. **Parallel execution is mandatory.** All 5 Agent calls in one function_calls block. This is the skill's reason for existence.
2. **Re-review after significant fixes.** If CRITICAL/MAJOR fixes were applied, the SAME 5 experts MUST re-review. Path: Phase 3 → Phase 4 → Phase 5 → Phase 6. If only MINOR/NITPICK found, re-review may be skipped.
3. **Same experts throughout.** The 5 experts assembled in Phase 2 are the SAME team for every round. Do not swap or regenerate experts.
4. **Diversity > depth.** 5 different review axes beat 5 experts in the same field.
5. **User confirms significant fixes.** CRITICAL/MAJOR fixes require user approval before applying. MINOR can be auto-applied. Never delete files, remove functionality, or make architectural changes without explicit approval.
6. **Validate after fixing.** Run build/tests after code fixes. Revert any fix that breaks tests.
7. **Prevent infinite loops.** Maximum 3 rounds. If unresolved by then, report as INCOMPLETE and stop.
8. **Final output in tables.** Fixes, expert verdicts, and issue tracking must be scannable at a glance.
