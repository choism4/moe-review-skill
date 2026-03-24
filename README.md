# Mob Review

Five experts. Five perspectives. One deliverable. Zero blind spots.

## Install

```bash
npx skills add choism4/mob-review
```

## Usage

```
/mob-review
```

Run it after completing any work — code, paper, report, design doc. The skill analyzes your deliverable, assembles a mob of 5 domain experts, and runs them in parallel. Fixes are applied automatically, and the loop repeats until every expert approves.

## What It Does

Mob Review assembles 5 virtual experts to mob your work from different angles simultaneously — catching issues that any single perspective would miss. Inspired by mob programming, where the whole team swarms a problem at once.

```
/mob-review
     │
     ▼
Phase 1: Analyze Deliverable
├── Code: git diff, identify languages/frameworks/domain
├── Docs: read files, identify type/audience/purpose
│
Phase 2: Assemble Expert Team (automatic)
├── Select 5 experts across different axes
├── Assign specific personas (name, company, experience)
│
Phase 3: Parallel Review                    ◄── THE CORE
├── Spawn 5 Agents simultaneously
├── Each reviews from their unique perspective
├── Returns structured JSON feedback
│
Phase 4: Synthesize + Auto-Fix
├── Consolidate feedback, prioritize by consensus
├── Auto-fix CRITICAL and MAJOR issues
├── Present summary table
│
Phase 5: Re-Review Loop (automatic)
├── Re-spawn 5 Agents with fix context
├── Repeat until all approve or 3 rounds reached
│
Phase 6: Final Report
└── Expert verdicts, fix summary, statistics
```

## Expert Selection

Experts are chosen to maximize perspective diversity:

**For code:**

| Axis | Example |
|------|---------|
| Architecture / Design | System design veteran |
| Domain Expertise | Field-specific practitioner |
| Code Quality | Clean code advocate, OSS maintainer |
| Security / Reliability | Security engineer, SRE |
| Performance / Optimization | Low-level specialist |
| Testing / QA | TDD practitioner |

**For documents:**

| Axis | Example |
|------|---------|
| Subject Expertise | Scholar or practitioner |
| Methodology / Logic | Research methods expert |
| Data / Numbers | Statistician, fact-checker |
| Style / Readability | Technical writer, editor |
| Target Audience | Actual reader persona |

### Example Teams

- **NLE code** → ex-Adobe Premiere engineer, Pixar graphics engineer, Netflix streaming architect, FFmpeg contributor, Apple FCP QA lead
- **NeurIPS paper** → DeepMind senior researcher, NeurIPS PC veteran, Stanford stats professor, Meta FAIR engineer, former Nature editor
- **Strategy report** → McKinsey partner, regional market specialist, Goldman Sachs analyst, The Economist editor, industry C-level

## Issue Severity

| Level | Meaning | Action |
|-------|---------|--------|
| CRITICAL | Must fix — bugs, security holes, factual errors | Auto-fixed immediately |
| MAJOR | Strongly recommend — design flaws, weak arguments | Auto-fixed immediately |
| MINOR | Improvement — naming, style, phrasing | Fixed at coordinator's judgment |
| NITPICK | Optional — personal preference | Usually ignored |

## Termination

| Condition | Result |
|-----------|--------|
| All 5 experts approve | Success |
| 0 CRITICAL/MAJOR issues | Success |
| 3 rounds reached | Report remaining issues and stop |
| No new issues (repeats only) | Stop |

## Deadlock Handling

- Conflicting expert opinions → majority rules, or higher confidence score wins
- Same issue flagged 2 rounds straight → escalated to user

## Example Output

```markdown
## Mob Review Complete

### Expert Team
| Expert | Specialty | Verdict |
|--------|----------|---------|
| Jun Park (ex-Cloudflare) | Security | ✅ APPROVED |
| Sujin Kim (ex-Stripe) | API Design | ✅ APPROVED |
| Alex Chen (ex-Google) | Performance | ✅ APPROVED |
| Maria Santos (ex-DataDog) | Observability | ✅ APPROVED |
| Tom Wright (ex-Vercel) | DX / Testing | ✅ APPROVED |

### Statistics
- Rounds: 2
- Issues found: 7 (CRITICAL 1, MAJOR 3, MINOR 3)
- Issues fixed: 6
- Not applied: 1 (NITPICK, style preference)
```

## Philosophy

- **Parallel, not serial** — 5 agents at once. No bias from earlier reviews.
- **Diversity > depth** — Different axes catch different blind spots.
- **Specific personas > generic roles** — "Ex-Cloudflare WAF lead" reviews differently than "security expert."
- **Conservative auto-fix** — Clear bugs get fixed. Design calls go to you.
- **Bounded loops** — 3 rounds max. No infinite review cycles.

## License

MIT
