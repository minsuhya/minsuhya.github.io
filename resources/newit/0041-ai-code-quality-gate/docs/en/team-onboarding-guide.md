# AI Code Quality Gate — Team Onboarding Guide

> One-page orientation for engineers joining a repo that uses this gate system.

---

## Why This Gate Exists

AI coding tools increase PR throughput by 2–4×. That's the benefit.  
The risk is that the same tools bypass the human judgment that catches logic errors, context mismatches, and security edge cases — because the code *looks* correct even when it isn't.

This gate exists to decouple the two effects:  
**AI tools make your fastest engineers faster. The gate prevents them from making your weakest code weaker.**

Without a gate, teams typically see:
- Rubber-stamp reviews ("LGTM" on diffs that were never read)
- AI-generated code that passes linting but fails in production due to business logic errors
- Security vulnerabilities that look like idiomatic code to a tired reviewer

---

## The Three-Layer Workflow

Every PR goes through three layers automatically:

**Layer 1 — Static Analysis** (runs in ~2 min, fully automated)  
Gitleaks catches hardcoded secrets. Bandit/Semgrep catches known vulnerability patterns. Radon flags functions with cyclomatic complexity above 20. Coverage enforcement ensures tests weren't skipped.  
→ *Hard blocker on CRITICAL findings. Soft warning on everything else.*

**Layer 2 — LLM Pre-Review** (runs in ~1 min, fully automated)  
A cross-model LLM analyzes the diff semantically — logic errors, security patterns, N+1 queries, architectural mismatches. Results post as a structured PR comment.  
→ *Hard blocker on CRITICAL. Human escalation on HIGH.*

**Layer 3 — Human Review** (you)  
You apply the 25-point checklist in the PR template. You focus on what automation cannot catch: business logic correctness, architectural fit, maintainability in the context of the full codebase.  
→ *Your judgment is the final gate. Use it on what matters.*

---

## The One Rule

> **The gate is a signal, not a blocker.**

Fast teams use signals faster. When Layer 1 or Layer 2 flags something, your job is not to dismiss it — it's to verify it quickly and either fix it or document why it's a false positive in the PR comments.

If the gate is slowing you down, the answer is not to bypass it — it's to refine the rules. Open an issue.

---

## What You Should Do on Your First PR

1. Check the `[x] This PR contains AI-generated code` box honestly — it helps calibrate the ruleset.
2. Read the Layer 2 comment before opening the diff. It gives you a 30-second brief on where to focus.
3. Work through the checklist in order. Skip items tagged `[AUTO]` if CI already passed.
4. If you disagree with a Layer 2 finding, add a one-line explanation in your review comment — don't just dismiss it.

---

## Common Questions

**"Does this slow down shipping?"**  
Layers 1 and 2 add ~3 minutes of wall-clock time. Layer 3 gets faster as reviewers internalize the checklist. The net effect after week 3 is faster shipping because production incidents drop.

**"What if CI is broken?"**  
Tag `@[team-lead]` and open an issue. Do not merge without gates running unless the lead explicitly approves — and document it.

**"The LLM flagged something that's fine — what do I do?"**  
Respond to the bot comment with a one-line explanation: `"False positive: this pattern is intentional because [reason]"`. This creates an audit trail and helps refine the prompt over time.

**"My PR is over 500 lines — Layer 2 skipped. Now what?"**  
Split the PR if possible. If not, ensure the domain owner reviews it directly. PRs over 500 lines have a statistically higher defect rate regardless of AI usage.
