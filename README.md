# AI-Assisted Engineering Playbook

> A practical playbook for engineering teams actively adopting AI coding tools (Copilot, Cursor, Codeium, etc.) in product engineering. This document outlines how to structure your codebase, define rules, manage context, and enforce testing to ensure AI accelerates delivery without degrading architecture, security, or quality.
>
> **Core Principle:** AI-generated code is never "done" just because it compiles. It is only done when deterministic tools, tests, and human review confirm its behavior. AI accelerates typing and scaffolding, but it does not replace engineering discipline.

---

## 1. Core Principles

1. **AI-readable, not AI-distorted.** The codebase must be understandable for both AI and humans. Do not break domain models just for the sake of model retrieval.
2. **Tests before scale.** Active AI coding requires a deterministic safety net: compile, unit, integration, e2e, lint, and security checks.
3. **Small bounded changes.** A single AI task must have a clear scope, explicit non-goals, and a test plan.
4. **Rules over vibes.** Formalize style, architectural invariants, security constraints, and review gates in repository rules.
5. **Least privilege context.** The model should only see what is strictly necessary for the task at hand.
6. **Separate roles.** Planning, implementation, review, and incident debugging should be separated by roles or dedicated AI sessions.
7. **Escalate early.** If a smaller model fails after 2-3 iterations, open a new context window and escalate to a stronger reasoning model.
8. **No silent authority.** AI does not make architectural decisions without an explicit plan and human review.
9. **Verification is mandatory.** AI-generated code is not done until tests pass and human review is complete.
10. **Cost is a product constraint.** Token budget, context size, and model selection must be managed like any other engineering resource.

---

## 2. Make the Codebase AI-Readable

AI coding tools retrieve context via keyword search, embeddings, file names, symbol names, and recently opened files. A codebase that is easy for a human to navigate is generally easier for an AI to understand.

### Practices
* Use unique, domain-specific class and object names.
* Name components by their bounded context: `RuntimeSessionRepository`, `ToolApprovalPolicy`, `BillingOrgService`.
* Maintain a strict Single Responsibility Principle per file.
* Add short module `README.md` files for large bounded contexts.
* Maintain a predictable package structure.

### Anti-Patterns
* Renaming everything solely for the AI, destroying established domain language.
* Keeping massive "god files" that mix API, persistence, validation, policy, and UI logic.
* Using generic names that confuse model searches (e.g., dozens of generic `DataService`, `Helper`, `Common`, or `Processor` files).
* Hiding critical business rules in randomly placed private methods without explicit naming.

---

## 3. Context and Data Boundaries

An AI tool does not need to see your entire monorepo, production logs, secrets, or customer data just because it is technically possible.

### Practices
* Configure strict ignore rules (`.cursorignore`, etc.) for `.env`, credentials, dumps, production logs, customer exports, and generated artifacts.
* Restrict context to the relevant module or bounded context.
* Use redacted logs and synthetic fixtures in prompts instead of real customer payloads.
* Perform a context inventory before starting a large task: explicitly define which files are needed and which are not.

### Cost & Quality Controls
* Do not keep massive debugging transcripts open after the hypothesis has changed. Open a fresh session with a compact context.
* Do not open large files unnecessarily.
* Do not include generated code in the context unless the task is to modify the generator.

---

## 4. Rules Architecture

Repository rules (e.g., `.cursorrules`) should be concise at the root level and detailed only where necessary.

### The Root Rule
The root rule applies to every prompt and must be extremely concise to save context and maintain focus. It should only contain:
* Security and multi-tenancy invariants.
* Hard architecture boundaries.
* A strict "no opportunistic refactoring" policy.
* Testing expectations.
* Commit/PR discipline.

### Module / Role Rules
Keep detailed rules in separate files invoked only when needed:
* Backend / Frontend framework guidelines.
* Database migration standards.
* Security review checklists.
* QA / Testing standards.

---

## 5. The AI SDLC (Software Development Life Cycle)

AI-assisted development works best as a pipeline, not as a single, endless chat thread.

### 1. Task Creation
* **Goal:** Turn a raw idea into a bounded task.
* **Output:** A task file containing status, context, requirements, non-goals, and impacted modules.

### 2. Technical Planning
* **Goal:** A strong reasoning model analyzes the task and creates an execution plan.
* **Output:** A plan containing objectives, likely touched files, API changes, security/migration risks, and a test plan.

### 3. Implementation
* **Goal:** A medium/coding model executes the exact plan.
* **Rules:** Execute the plan only. No scope expansion. No opportunistic refactoring. No silent dependency upgrades. Stop and ask for clarification if the plan is flawed.

### 4. Review and Cleanup
* **Goal:** The AI reviewer does not just "polish" code; it looks for bugs, regressions, security gaps, and missing tests.

### 5. Final Verification (Deterministic)
* **Goal:** Hard gates before commit/PR (Compile, Unit/Integration/E2E tests, Linting, Security checks).

---

## 6. Role Separation

Do not give a single AI session the freedom to plan, write, review, and debug endlessly.

| Role | Model Class | Responsibility |
|---|---|---|
| **Task Creator** | Strong general | Turn raw ideas into bounded tasks. |
| **Planner** | Strong reasoning | Draft technical plans, risks, and test strategies. |
| **Context Delegate** | Cheap/Fast | Gather relevant files and summarize context. |
| **Implementer** | Medium coding | Execute the bounded plan precisely. |
| **Reviewer** | Medium/Strong | Find bugs, regressions, and missing tests. |
| **QA Engineer** | Coding/Test | Add deterministic tests (unit, integration). |
| **Responder** | Strong/Debug | Produce minimal root-cause fixes for incidents. |

---

## 7. The Context Delegate Pattern

To save costs and improve focus, use a fast/cheap model to gather context before executing complex tasks.

1. The Context Delegate gathers relevant files, symbols, constraints, and open questions.
2. The Delegate writes the summary to a temporary file (e.g., `docs/tmp/task-context.md`).
3. The Strong model uses this curated file for planning or architecture analysis, rather than scanning the entire repository.

---

## 8. AI Change Budgets

AI often writes more code than necessary. This must be constrained.

### Default Limits
* One task = one bounded outcome.
* No unrelated refactoring.
* No dependency upgrades unless explicitly required.
* No architectural changes buried inside a bugfix.
* No silent public API changes.
* If more files need changes than expected, the AI must stop and explain why.

---

## 9. Testing and Verification Ladder

The more code AI writes, the stronger your automated verification must be. AI-generated code is not done because it "looks right."

**Before heavy AI adoption:**
* Elevate compile/typecheck to a reliable baseline.
* Cover core business logic with unit tests.
* Add integration tests for persistence, API boundaries, auth, and webhooks.
* Remove or isolate flaky tests.

**After every AI change:**
1. Compile / Typecheck.
2. Unit tests for touched logic.
3. Integration tests for persistence/API changes.
4. Lint / Format.

---

## 10. AI Code Review Checklist

When reviewing AI-generated code, humans (and AI reviewers) must look for production risks, not just stylistic preferences:

* Does the change exactly match the task/plan?
* Are there unrelated edits?
* Are architectural boundaries respected?
* Are all SQL/DB paths safely tenant-scoped?
* Are there PII or secrets leaking into logs, traces, or tests?
* Is the changed behavior covered by tests?
* Was an unnecessary dependency added?
* Is there rollback or migration risk?
* Does error handling swallow the actual root cause?

---

## 11. Debugging and Escalation Rules

AI debugging can easily devolve into an infinite loop of breaking and fixing.

* Stop the AI after 2-3 failed iterations.
* Open a new session with a clean, compact context.
* Formulate the exact failure, logs, and a specific hypothesis.
* Escalate to a stronger reasoning model for root-cause analysis.
* Do not allow the model to randomly mutate unrelated files to "see if it works."
* Always add a regression test after the fix.

---

## 12. Example Operating Manual (Prompts)

Use specific, role-based prompts combined with markdown context files.

**1. Task Creation**
> `@task-creator.mdc` Add a task: Integrate Stripe webhooks to update organization subscription statuses. Priority P2.

**2. Planning**
> `@planner.mdc` Analyze `@docs/todo/my-task.md` and create an execution plan in `docs/todo/in_progress/task.md`. If anything is ambiguous, ask clarifying questions and propose recommended options.

**3. Implementation**
> Execute `docs/todo/in_progress/task.md`. Do not expand the scope.

**4. Context Delegation**
> `@context-delegate.mdc` Gather context for the following idea: asynchronous external callbacks for our FSM. Write the summary to `docs/tmp/fsm-callbacks-context.md`.

**5. QA / Testing**
> `@qa-engineer.mdc` Write Testcontainers-backed tests for `@RuntimeService.scala`. Ensure they are deterministic and cover edge cases.

**6. Incident Response**
> `@bug-hunter.mdc` Here is a production log: [LOG]. Find the root cause, explain it in one sentence, and apply the most minimal fix possible without refactoring.

---

## 13. Team Governance & Metrics

AI adoption must be a team process, not individual magic. If code output increases but review time, defect rates, and rollback rates also increase, AI adoption has not improved delivery—it has merely shifted the cost from writing to debugging.

**Track these metrics:**
* Cycle time per task.
* PR review time.
* Defect rate after merge / Escaped bugs.
* Flaky tests introduced.
* Number of AI iterations before success (measure loop waste).

---

*This playbook is maintained by the engineering team at **[PlanVault](https://planvault.ai)** — an event-sourced execution layer for AI agents. We developed these practices while building our core engine to ensure high-velocity AI assistance doesn't compromise architecture, security, or testing discipline. If your team is struggling with AI governance in production, feel free to adopt these workflows or check out our platform.*
