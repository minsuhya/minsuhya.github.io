## AI Code Review Checklist

> **Gate context**: Layer 1 (static analysis) and Layer 2 (LLM pre-review) run automatically in CI.  
> This checklist directs your human attention to what automation cannot catch.

### AI-Generated Code Flag

- [ ] This PR contains AI-generated code (Copilot / Cursor / ChatGPT / other AI tool)

---

### Layer 3: Human Review — 25-Point Checklist

> Items tagged `[AUTO]` are primarily caught by Layers 1–2. Review them only if CI did not run.

#### 1. Correctness & Logic (8 items) · *detection difficulty: HIGH*

- [ ] **C1** All error-handling paths cover exception types the AI may have silently ignored
- [ ] **C2** Null, empty, and boundary inputs are explicitly handled — not just the happy path
- [ ] **C3** Algorithmic complexity matches what the system actually needs under real load
- [ ] **C4** The logic was traced end-to-end, not just read — edge cases were mentally executed
- [ ] **C5** No off-by-one errors in loop bounds, slice indices, or pagination offsets
- [ ] **C6** Async/concurrency patterns are correct: no race conditions, no deadlocks, no missed awaits
- [ ] **C7** Return values and error signals are propagated correctly up the entire call stack
- [ ] **C8** Business logic matches the ticket acceptance criteria — not just "plausible" behavior

#### 2. Security & Trust (6 items) · *detection difficulty: MEDIUM — Layers 1–2 catch some*

- [ ] **S1** Input validation exists server-side on all public-facing entry points `[AUTO partial]`
- [ ] **S2** Output encoding applied before rendering any user-supplied content
- [ ] **S3** No hardcoded credentials, API keys, or environment-specific values `[AUTO]`
- [ ] **S4** No privilege escalation paths — AI-generated auth logic gets extra scrutiny
- [ ] **S5** SQL / command / path injection vectors are neutralized `[AUTO partial]`
- [ ] **S6** Sensitive data (PII, tokens, passwords) is not logged or exposed in error messages

#### 3. Context & Integration (6 items) · *detection difficulty: HIGH — primary human review zone*

- [ ] **I1** Code fits the architectural pattern of the surrounding system — no foreign patterns inserted
- [ ] **I2** Existing shared utilities and abstractions are reused — no silent re-implementations
- [ ] **I3** No breaking changes to public APIs, shared interfaces, or event contracts
- [ ] **I4** Database queries respect existing transaction and locking patterns
- [ ] **I5** No N+1 query patterns introduced — AI generates these without hesitation
- [ ] **I6** Module dependencies match the current migration or refactor state of the codebase

#### 4. Maintainability & Ownership (5 items) · *detection difficulty: HIGH*

- [ ] **M1** A mid-level engineer can understand and debug this code at 2am without the author present
- [ ] **M2** Naming matches the codebase vocabulary — no foreign naming dialect injected by the AI
- [ ] **M3** No over-abstraction: no unnecessary layers of interfaces wrapping trivial logic
- [ ] **M4** Test coverage was updated meaningfully — not just added; existing tests still assert correctly
- [ ] **M5** The team's established code style, patterns, and architectural conventions are respected

---

### Escalation Decision

- [ ] **MERGE** — all applicable items verified, no blockers
- [ ] **REQUEST CHANGES** — specific items flagged (add line references in review comments)
- [ ] **ESCALATE TO SENIOR** — touches auth / payment / PII / shared infrastructure
