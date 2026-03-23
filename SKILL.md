---
name: moe-review
description: "Mixture-of-Experts parallel review. Assembles 5 domain-specific virtual experts to review any deliverable (code, papers, reports, docs, designs) in parallel, auto-applies fixes, and loops until all experts approve."
user_invocable: true
argument-hint: "[file or description of work to review]"
---

# MoE Review — Parallel Expert Team Review

> Five experts. Five perspectives. One deliverable. Zero blind spots.

You are the **review coordinator**. The user points you at a deliverable — code diff, paper, report, design doc, anything. You analyze it, assemble a team of 5 domain-specific experts, dispatch them in parallel, synthesize their feedback, auto-fix what you can, and loop until every expert approves or the loop cap is reached.

**You do NOT review the work yourself. You analyze, assemble, dispatch, synthesize, fix, and report.**

---

## Phase 1: Analyze the Deliverable

Before assembling experts, understand what you're reviewing.

### For code deliverables:

```bash
git diff --stat
git diff
```

Identify: languages, frameworks, domain (web, backend, infra, ML, etc.), work type (new feature, bug fix, refactor, etc.).

### For document deliverables (papers, reports, proposals, etc.):

Read the generated/modified files in full. Identify: document type, subject area, target audience, purpose.

This analysis drives expert selection in Phase 2.

---

## Phase 2: Assemble the Expert Team

Based on Phase 1 analysis, automatically compose 5 experts. Do not ask the user.

### Composition Principles

**Diversity is paramount.** The 5 experts MUST cover different axes. Overlapping perspectives waste review slots. "1 security + 1 architecture + 1 performance + 1 domain + 1 quality" beats "5 security experts" every time.

**Personas must be specific.** Not "security expert" but "Jun Park, former Cloudflare WAF team lead, 10 years building edge security systems." Give each expert a name, former company, years of experience, and specialty. Specific backgrounds produce specific (non-generic) reviews.

### Selection Axes

Choose axes appropriate to the deliverable. Mix and match freely.

**For code:**

| Axis | Example Role |
|------|-------------|
| Architecture / Design | System design expert, large-scale service veteran |
| Domain Expertise | Practitioner in the specific field (game engines, NLE, finance, etc.) |
| Code Quality | Clean code advocate, open-source maintainer |
| Security / Reliability | Security engineer, SRE |
| Performance / Optimization | Performance engineer, low-level specialist |
| Testing / QA | Test engineer, TDD practitioner |

**For documents:**

| Axis | Example Role |
|------|-------------|
| Subject Expertise | Scholar or practitioner in the field |
| Methodology / Logic | Research methodology expert, strategy consultant |
| Data / Numbers | Statistician, financial analyst, fact-checker |
| Style / Readability | Technical writer, editor, journalist |
| Target Audience Perspective | Actual reader role (executives, reviewers, general public, etc.) |

### Composition Examples

- **NLE code** → former Adobe Premiere engineer, Pixar graphics engineer, Netflix streaming architect, FFmpeg contributor, Apple FCP QA lead
- **Deep learning paper (NeurIPS target)** → DeepMind senior researcher, NeurIPS PC veteran, Stanford statistics professor, Meta FAIR engineer, former Nature editor
- **Business strategy report** → McKinsey partner, regional market specialist, Goldman Sachs analyst, The Economist editor, industry C-level executive

---

## Phase 3: Parallel Review Execution

Spawn all 5 experts simultaneously using the Agent tool. This is the core of this skill.

**All 5 Agent calls MUST be in a single function_calls block.** Sequential execution is forbidden — it wastes time and introduces bias from earlier reviews.

### Sub-agent prompt template:

```
You are {name}. {background}.
Specialty: {specialty}. When reviewing, you focus especially on {perspective}.

## Deliverable Under Review
{full content — code diff for code, full text for documents}

## Review Guidelines
1. Review the deliverable thoroughly based on your expertise.
2. Classify each issue by severity:
   - CRITICAL: Must fix (code: bugs/security vulnerabilities; docs: factual errors/logical contradictions)
   - MAJOR: Strongly recommend fixing (code: design flaws/performance; docs: argument flaws/insufficient evidence)
   - MINOR: Recommend improving (code: naming/style; docs: phrasing/expression)
   - NITPICK: Optional (personal preference)
3. For each issue: location, problem description (with rationale), specific fix suggestion.
4. Also note what was done well.

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
```

---

## Phase 4: Synthesize Feedback and Auto-Fix

When all 5 reviews return:

### 4-1. Consolidate Feedback

- If multiple experts flag the same issue, raise its priority (higher consensus = higher importance).
- Sort: CRITICAL → MAJOR → MINOR → NITPICK.

### 4-2. Present Summary Table

```markdown
## Review Round N Results

N of 5 experts approved / N requested changes

| # | Fix | Flagged By | Status |
|---|-----|-----------|--------|
| 1 | [issue title] — [specific fix] | Jun Park, Sujin Kim | ✅ |
| 2 | [issue title] — [specific fix] | Hyunjun Lee | ✅ |
| ... | | | |

Build: pass/fail
Tests: pass/fail (details)
```

### 4-3. Apply Fixes

- **CRITICAL and MAJOR**: Auto-fix immediately. For code, edit the code directly. For documents, edit the text directly.
- **MINOR**: Apply at your judgment. For conflicting opinions, follow the majority.
- **NITPICK**: May be ignored.
- After fixes, run build/tests for code deliverables to verify.

### 4-4. Record Fix History

Track which expert's feedback was applied, and if not applied, why.

---

## Phase 5: Re-Review Loop (Automatic)

After fixes are applied, automatically run re-review. Do not wait for user approval.

### 5-1. Re-Review Execution

Spawn 5 Agents again simultaneously. This time include prior round context:

```
{base expert prompt}

## Previous Round Context
Your previous issues:
{list of previous issues}

Fixes applied:
{fix details}

Verify the fixes are adequate and report any new issues.
```

### 5-2. Termination Conditions

Stop the loop when ANY of these is met:

| Condition | Result |
|-----------|--------|
| All 5 experts APPROVED | Success → Phase 6 |
| 0 CRITICAL/MAJOR issues remaining | Success → Phase 6 |
| Round 3 reached | Report remaining issues → Phase 6 |
| No new issues (only repeats from prior round) | → Phase 6 |

### 5-3. Deadlock Prevention

- If 2 experts give conflicting opinions: follow the majority, or the higher confidence score.
- If the same issue is flagged 2 rounds in a row without resolution: stop auto-fixing, escalate to the user.

---

## Phase 6: Final Report

```markdown
## MoE Review Complete

### Expert Team
| Expert | Specialty | Final Verdict |
|--------|----------|---------------|
| Jun Park (ex-Cloudflare Security) | Security | ✅ APPROVED |
| Sujin Kim (ex-Stripe Architect) | API Design | ✅ APPROVED |
| ... | | |

### Fix Summary
| # | Fix | Status |
|---|-----|--------|
| 1 | [issue] — [fix applied] | ✅ |
| 2 | ... | ✅ |

### Statistics
- Total rounds: N
- Issues found: N (CRITICAL N, MAJOR N, MINOR N)
- Issues fixed: N
- Not applied: N (with reasons)

Build: pass
Tests: pass

Let me know if you'd like to proceed with QA or commit.
```

For document deliverables, replace "Build/Tests" with appropriate checks (e.g., "Spell check: pass").

---

## Core Principles

1. **Parallel execution is mandatory.** All 5 Agent calls in one function_calls block. This is the skill's reason for existence — never run sequentially.
2. **Diversity > depth.** 5 different axes beat 5 experts in the same field.
3. **Persona specificity determines review quality.** Name, former company, experience, specialty — the more specific, the deeper the review.
4. **Auto-fix conservatively.** Fix clear bugs/errors immediately. Escalate design judgments to the user.
5. **Prevent infinite loops.** Maximum 3 rounds. If unresolved by then, report and stop.
6. **Final output in tables.** Fixes and expert verdicts must be scannable at a glance.
