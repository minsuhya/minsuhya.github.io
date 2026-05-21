# Do-Not-Merge vs Fast-Track-Safe Reference Card

> Print this. Keep it visible during code review.  
> The difference between teams that survive AI-assisted development and teams that don't is whether this list is a reflex.

---

## 🔴 Do-Not-Merge — 10 Patterns That Require Immediate Escalation

No exceptions. If any of these are present, stop and escalate to a senior reviewer.

| # | Pattern | Why it's dangerous |
|---|---------|-------------------|
| 1 | **Cyclomatic complexity > 20** in a single function | AI generates deeply nested logic that technically works but is unmaintainable and untestable |
| 2 | **Empty catch blocks** or bare exception swallowing | Silent failure in production; errors disappear without trace |
| 3 | **Hardcoded credentials**, connection strings, or environment-specific values | Instant security incident if merged to a public repo or deployed |
| 4 | **Generic variable names** (`data`, `result`, `temp`, `obj`) across 30+ lines | AI naming is a readability time bomb; no one can debug this at 2am |
| 5 | **Missing null checks** on external API responses | AI assumes happy path; production APIs return unexpected nulls regularly |
| 6 | **Synchronous calls inside loops** (N+1 pattern) | AI generates these without hesitation; they pass tests and destroy production under load |
| 7 | **Missing input validation** on any public-facing function | The most common AI security gap; validation often appears only client-side |
| 8 | **Deprecated library usage** (AI trained on outdated docs) | May work today, breaks silently on next dependency update |
| 9 | **Over-abstraction** — 3 layers of interfaces for a 10-line utility | AI loves defensive over-engineering; it decouples intent from implementation invisibly |
| 10 | **No tests**, or tests that only assert `true == true` | AI writes tests that pass without testing anything meaningful |

---

## ✅ Fast-Track-Safe — 10 Green Signals

If all ten of these are present, the PR can move through review quickly.

| # | Signal | What it tells you |
|---|--------|------------------|
| 1 | **PR under 200 lines** | Reviewable in one focused session; defect density correlates with PR size |
| 2 | **Single responsibility** — one logical change, one domain | No hidden coupling; rollback is clean |
| 3 | **Existing test coverage updated** | The author verified the change, not just the AI |
| 4 | **No new dependencies added** | No supply chain risk introduced |
| 5 | **No security-adjacent code touched** | Auth, payments, PII, sessions not in scope |
| 6 | **LLM pre-reviewer returned clean** (Layer 2 passed) | Semantic analysis found no risk signals |
| 7 | **Domain owner has reviewed** | Someone who owns this part of the codebase confirmed it fits |
| 8 | **All linter checks pass** (Layer 1 passed) | No known vulnerability patterns, no secret leaks |
| 9 | **Ticket acceptance criteria verifiably met** | The PR does what the ticket asked — confirmed, not assumed |
| 10 | **Rollback path is obvious** | If this breaks production, reverting is one command with no side effects |

---

## How to Use This Card

- **Reviewer**: Run the Do-Not-Merge list mentally in the first 60 seconds before reading code. If you spot any of the 10 patterns, stop and escalate immediately rather than reading further.
- **Author**: Before opening the PR, verify the Fast-Track-Safe list. If fewer than 7 of 10 apply, reconsider whether the PR is ready for review.
- **Team lead**: If the same Do-Not-Merge patterns keep appearing, that's a signal to add a targeted automated rule to Layer 1 — not to keep catching it manually.
