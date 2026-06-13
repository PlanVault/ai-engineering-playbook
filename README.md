# AI-Assisted Engineering Playbook

> A practical playbook for engineering teams actively adopting AI coding tools (Copilot, Cursor, Claude, etc.) in product engineering. This document outlines how to structure your codebase, manage knowledge, define rules, manage context, and enforce testing to ensure AI accelerates delivery without degrading architecture, security, or quality.
>
> **Core Principle:** AI-generated code is never "done" just because it compiles. It is only done when deterministic tools, tests, and human review confirm its behavior. AI accelerates typing and scaffolding, but it does not replace engineering discipline.

---

## Quick Start

1. **Copy the foundation rules** into `.cursor/rules/` (or your AI tool's rules folder). These two apply to every session automatically:
   - [`root-rule.mdc`](example_rules/root-rule.mdc) — universal security, multi-tenancy, and scope guard
   - [`change-budget.mdc`](example_rules/change-budget.mdc) — AI change budget enforcer with stop conditions

2. **Add your stack rules** — pick the files that match your backend and frontend:
   - `python-backend.mdc`, `java-backend.mdc`, `go-backend.mdc`, `javascript-backend.mdc`, `scala-backend.mdc`
   - `react-frontend.mdc`

3. **Add process rules** for the AI SDLC pipeline:
   - `task-creator.mdc` → `planner.mdc` → *(implementer uses stack rule)* → `reviewer.mdc` → `qa-engineer.mdc`

4. **Add cross-cutting rules** as needed:
   - `database-migrations.mdc`, `api-design.mdc`, `observability.mdc`, `security-auditor.mdc`, `infra-devops.mdc`

> All rules are in [`example_rules/`](example_rules/). Each file is self-contained and can be copied into any project.

---

## 1. Core Principles

1. **Human-and-AI-readable.** Infrastructure, code, and documentation must work effectively for both humans and AI agents — not one at the expense of the other.
2. **Tests before scale.** Active AI coding requires a deterministic safety net: compile, unit, integration, e2e, lint, and security checks — set up before generating code at scale.
3. **Small bounded changes.** A single AI task must have a clear scope, explicit non-goals, and a test plan.
4. **Rules over vibes.** Formalize style, architectural invariants, security constraints, and review gates in repository rules.
5. **Granular context and tool access.** The model should only see the files and tools strictly necessary for the task at hand.
6. **Separate roles.** Planning, implementation, review, and incident debugging should be separated by roles or dedicated AI sessions.
7. **Escalate early.** If a smaller model fails after 2-3 iterations, open a new context window and escalate to a stronger reasoning model.
8. **No silent authority.** AI does not make architectural decisions without an explicit plan and human review.
9. **Verification is mandatory.** AI-generated code is not done until tests pass and human review is complete.
10. **Cost is a product constraint.** Token budget, context size, and model selection must be managed like any other engineering resource.
11. **Agentic engineering is a living process.** Models change, tools change, requirements change. What worked last quarter may not work today. Tune continuously, per project.

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

## 3. Knowledge Base Infrastructure

**The most impactful, most overlooked step.** Before you can effectively use AI agents at scale, you need a structured, machine-readable knowledge base that lives alongside your code.

### Core Idea
Convert your existing documentation — Confluence pages, Notion docs, Google Docs, internal wikis — into `.md` files placed near the code they describe. Agents read files directly; they don't need a RAG pipeline if your knowledge is well-organized.

### Structure
```
docs/
  architecture/       # System design, ADRs, bounded context maps
  api/                # API contracts, versioning decisions
  runbooks/           # Operational procedures
  onboarding/         # Team onboarding, coding standards
  reference/          # Domain glossary, third-party integrations
  todo/               # Active tasks (in_progress/, backlog/)
  tmp/                # Temporary context files for agents
knowledge/
  product/            # Product specs, business rules
  compliance/         # GDPR, security policies
```

### Rules
* Every top-level service or module gets a `README.md` explaining its purpose, boundaries, and key entry points.
* Add an `AGENTS.md` (or `README.md`) to each significant directory with a brief description for AI agents.
* Keep documentation close to the code it describes — not in a separate monolithic wiki.
* Update docs as part of the same PR that changes the code.

### RAG Is Not a Silver Bullet
RAG does not solve documentation debt. If your existing knowledge is scattered, contradictory, or outdated, a RAG pipeline will return scattered, contradictory, outdated answers. Fix the source first.

> **Rule of thumb:** If a new engineer can't find the answer in your `docs/` folder in 5 minutes, an AI agent can't find it reliably either.

---

## 4. Context and Data Boundaries

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

## 5. Rules Architecture

Repository rules (e.g., `.cursor/rules/`) should be concise at the root level and detailed only where necessary.

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

### Sharing Rules Across Tools
If your team uses different AI tools (some use Cursor, others Claude, others Copilot), write rules to be portable:
* Store canonical rules in `docs/rules/` or `.cursor/rules/`.
* Add an `AGENTS.md` file in each key directory — Claude and other agents pick these up automatically.
* Cursor's `.mdc` rules and Claude's `AGENTS.md` can reference the same underlying guidelines.
* Link to the shared rules from each tool-specific config so they stay in sync.

---

## 6. Granular Tool & Context Profiles

Do not give every agent session access to everything. Compose the minimum set of tools and files for the task.

### Why It Matters
* Excess context increases cost, latency, and the risk of hallucination.
* A test-writing agent does not need production DB access.
* A planning agent does not need filesystem write access.

### Example Profiles
| Role | Files | Tools |
|---|---|---|
| **Planner** | `docs/`, architecture files | `web-search`, `read` |
| **Implementer** | Relevant module only | `read`, `write`, `terminal` (tests only) |
| **Reviewer** | Changed files + test files | `read`, `git-diff` |
| **QA Engineer** | Test files + source under test | `read`, `write`, `test-runner` |
| **Context Delegate** | Entire repo | `read`, `search` (no write) |

### MCP / Integration Hygiene
* Enable MCP servers per-project, not globally.
* Disable integrations that are not needed for the current task category.
* Audit which tools are active before starting a long agentic run.

---

## 7. Meta Repository Pattern (Microservices)

When services live in separate repositories, no single AI agent can see the full picture. Cross-service changes become fragmented and error-prone.

### The Problem
* Agent writes code in Service A without knowing Service B's contract changed.
* Planning agent can't reason about end-to-end flows.
* No shared rules, no shared documentation.

### Solution: Meta Repository
Create a `meta` or `platform` repository that:
* Links to each service repo as a `git submodule` (or contains pointer files).
* Contains system-wide architecture docs, ADRs, and bounded context maps.
* Holds shared rules (`.cursor/rules/`, `AGENTS.md`) that individual repos reference.
* Documents the public API contract of each service.
* Becomes the entry point for any cross-service planning task.

```
meta-repo/
  README.md              # System overview
  docs/
    architecture/        # C4 diagrams, ADRs
    services/            # One file per service: purpose, API, owners
    contracts/           # Shared event schemas, API types
  .cursor/rules/         # Shared rules all services inherit
  AGENTS.md              # System-level context for AI agents
  services/
    service-a/           # git submodule or pointer
    service-b/
```

> **Practical tip:** Even without submodules, maintaining a `docs/services/` directory in the meta repo — with one `.md` per service describing its purpose, API surface, and dependencies — gives AI agents the cross-service context they need for planning.

---

## 8. The AI SDLC (Software Development Life Cycle)

AI-assisted development works best as a pipeline, not as a single, endless chat thread.

### 1. Task Creation
* **Goal:** Turn a raw idea into a bounded task.
* **Output:** A task file containing status, context, requirements, non-goals, and impacted modules.

### 2. Technical Planning
* **Goal:** A strong reasoning model analyzes the task and creates an execution plan.
* **Output:** A plan containing objectives, likely touched files, API changes, security/migration risks, and a test plan.

### 3. Spec File (Before Implementation)
* **Goal:** Before writing a single line of code, produce a spec: API contract, expected behavior, edge cases, and test cases.
* **Output:** A `spec.md` file alongside the task file.
* **Why:** The implementer reads the spec, not the chat history. The spec becomes the source of truth and the review checklist.

### 4. Implementation
* **Goal:** A medium/coding model executes the exact plan against the spec.
* **Rules:** Execute the plan only. No scope expansion. No opportunistic refactoring. No silent dependency upgrades. Stop and ask for clarification if the plan is flawed.

### 5. Review and Cleanup
* **Goal:** The AI reviewer does not just "polish" code; it looks for bugs, regressions, security gaps, and missing tests — checked against the original spec.

### 6. Final Verification (Deterministic)
* **Goal:** Hard gates before commit/PR (Compile, Unit/Integration/E2E tests, Linting, Security checks). Agents must be able to run these themselves and self-correct.

---

## 9. Role Separation

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

## 10. The Context Delegate Pattern

To save costs and improve focus, use a fast/cheap model to gather context before executing complex tasks.

1. The Context Delegate gathers relevant files, symbols, constraints, and open questions.
2. The Delegate writes the summary to a temporary file (e.g., `docs/tmp/task-context.md`).
3. The Strong model uses this curated file for planning or architecture analysis, rather than scanning the entire repository.

---

## 11. AI Change Budgets

AI often writes more code than necessary. This must be constrained.

### Default Limits
* One task = one bounded outcome.
* No unrelated refactoring.
* No dependency upgrades unless explicitly required.
* No architectural changes buried inside a bugfix.
* No silent public API changes.
* If more files need changes than expected, the AI must stop and explain why.

---

## 12. Testing and Verification Ladder

**Set up your test infrastructure before you start generating code at scale.** Without it, agents cannot verify their own output, and you lose the feedback loop that makes AI development safe.

### Before Heavy AI Adoption (Non-Negotiable)
* CI pipeline must enforce: `lint → typecheck → unit tests → integration tests` with hard failure on any step.
* Cover core business logic with unit tests.
* Add integration tests for persistence, API boundaries, auth, and webhooks.
* Remove or isolate flaky tests — flaky tests destroy agent self-correction.
* Agents must be able to run the full test suite locally and interpret the output.

### After Every AI Change
1. Compile / Typecheck.
2. Unit tests for touched logic.
3. Integration tests for persistence/API changes.
4. Lint / Format.

### Self-Correcting Agents
When agents can run tests themselves, they can iterate to green without human intervention. This only works if:
* Tests are deterministic (no random seeds, no time-dependent behavior, no magic sleeps).
* Test output is human-readable and parseable (clear failure messages, not stack dumps).
* The test command is documented in `README.md` or a `Makefile`.

---

## 13. Mandatory CI/CD Gates

Automate review-time checks before the team starts generating code at volume. These gates are the safety net for everything above.

### Required Gates (Block Merge if Failing)
* **Lint & Format** — `eslint`, `ruff`, `golangci-lint`, etc.
* **Typecheck** — `tsc --noEmit`, `mypy`, etc.
* **Unit Tests** — all must pass, coverage threshold enforced.
* **Integration Tests** — DB, API, auth boundaries.
* **Security Scan** — `trivy`, `semgrep`, `tfsec` for infra changes.
* **Conventional Commits** — enforced by CI, not by honor system.

### Why This Must Come First
AI agents write code faster than humans can review it. Without automated gates, defects accumulate invisibly. With them, agents self-correct before the PR even opens.

---

## 14. AI Code Review Checklist

When reviewing AI-generated code, humans (and AI reviewers) must look for production risks, not just stylistic preferences:

* Does the change exactly match the task/plan/spec?
* Are there unrelated edits?
* Are architectural boundaries respected?
* Are all SQL/DB paths safely tenant-scoped?
* Are there PII or secrets leaking into logs, traces, or tests?
* Is the changed behavior covered by tests?
* Was an unnecessary dependency added?
* Is there rollback or migration risk?
* Does error handling swallow the actual root cause?

---

## 15. Debugging and Escalation Rules

AI debugging can easily devolve into an infinite loop of breaking and fixing.

* Stop the AI after 2-3 failed iterations.
* Open a new session with a clean, compact context.
* Formulate the exact failure, logs, and a specific hypothesis.
* Escalate to a stronger reasoning model for root-cause analysis.
* Do not allow the model to randomly mutate unrelated files to "see if it works."
* Always add a regression test after the fix.

---

## 16. Model Benchmarking and Selection

Different models have different strengths. Periodic evaluation on your real tasks is the only reliable way to know which model to use for what.

### Benchmark Process
* **Frequency:** Monthly or after a significant model release.
* **Task categories:** Planning, code generation, refactoring, writing tests, SQL, debugging, documentation.
* **Measure:** Number of iterations to success, % of PRs accepted without revision, review time, escaped bugs.

### Practical Guidance
* Strong reasoning models (o3, Claude Opus) → planning, architecture, cross-service analysis.
* Medium coding models (Claude Sonnet, GPT-4o) → implementation, refactoring.
* Fast/cheap models (Haiku, GPT-4o-mini) → context gathering, formatting, simple lookups.
* Do not use a reasoning model for trivial completions — the cost and latency are unjustified.
* Track results in a shared doc or spreadsheet so the team learns collectively, not individually.

---

## 17. Example Rules Library

All rules live in [`example_rules/`](example_rules/). Copy the ones you need into `.cursor/rules/`.

### Foundation — always active
| File | Trigger | Purpose |
|---|---|---|
| [`root-rule.mdc`](example_rules/root-rule.mdc) | `alwaysApply: true` | Universal security, multi-tenancy, scope guard, and testing gate |
| [`change-budget.mdc`](example_rules/change-budget.mdc) | `alwaysApply: true` | Prevents scope creep; defines STOP conditions and self-check before finalizing |

### Process & Planning
| File | Trigger | Purpose |
|---|---|---|
| [`task-creator.mdc`](example_rules/task-creator.mdc) | `docs/todo/README.md` | Turns raw ideas into structured ADR task files |
| [`planner.mdc`](example_rules/planner.mdc) | `docs/todo/in_progress/*.md` | Converts ADRs into strict technical execution contracts |
| [`context-delegate.mdc`](example_rules/context-delegate.mdc) | `docs/tmp/*-context.md` | Gathers codebase context and generates prompts for external LLMs |
| [`reviewer.mdc`](example_rules/reviewer.mdc) | `*` | Cross-boundary code review, consistency checks, and cleanup |
| [`pr-standards.mdc`](example_rules/pr-standards.mdc) | `.github/**` | Conventional Commits, atomic PR rules, and merge gates |
| [`business-seo-analyst.mdc`](example_rules/business-seo-analyst.mdc) | `docs/ideas/*.md` | Breaks large product ideas into Phase 1 / Phase 2 scopes |

### Language — Backend
| File | Trigger | Purpose |
|---|---|---|
| [`python-backend.mdc`](example_rules/python-backend.mdc) | `**/*.py` | FastAPI/Django/Flask: async, mypy strict, parameterized SQL, typed errors |
| [`java-backend.mdc`](example_rules/java-backend.mdc) | `**/*.java` | Spring Boot/Quarkus: Optional, typed exceptions, records, PreparedStatement |
| [`go-backend.mdc`](example_rules/go-backend.mdc) | `**/*.go` | Explicit error wrapping, context propagation, errgroup, table-driven tests |
| [`javascript-backend.mdc`](example_rules/javascript-backend.mdc) | `**/*.ts, **/*.js` | Node.js/Express/NestJS: strict TS, zod env validation, typed error middleware |
| [`scala-backend.mdc`](example_rules/scala-backend.mdc) | `**/*.scala` | Cats Effect IO, Doobie fragments, sealed trait errors, parMapN |

### Language — Frontend
| File | Trigger | Purpose |
|---|---|---|
| [`react-frontend.mdc`](example_rules/react-frontend.mdc) | `**/*.tsx, **/*.ts` | React 18 + TypeScript: TanStack Query, strict types, i18n, RFC 7807 errors |
| [`ui-ux-designer.mdc`](example_rules/ui-ux-designer.mdc) | `**/*.tsx` | Accessibility (a11y), visual hierarchy, design system consistency |

### Infrastructure & Data
| File | Trigger | Purpose |
|---|---|---|
| [`infra-devops.mdc`](example_rules/infra-devops.mdc) | Terraform, Docker, CI/CD files | Cloud infra, secrets management, trivy/tfsec security scan |
| [`database-migrations.mdc`](example_rules/database-migrations.mdc) | SQL and migration files | Zero-downtime patterns, expand-contract, `CREATE INDEX CONCURRENTLY` |
| [`api-design.mdc`](example_rules/api-design.mdc) | Routes and controller files | REST naming, RFC 7807 errors, cursor pagination, idempotency keys |

### Quality & Observability
| File | Trigger | Purpose |
|---|---|---|
| [`qa-engineer.mdc`](example_rules/qa-engineer.mdc) | Test files | Deterministic tests, real DB containers, no magic sleeps |
| [`bug-hunter.mdc`](example_rules/bug-hunter.mdc) | `*` | Root-cause analysis from logs, minimal hotfix, structured logging |
| [`observability.mdc`](example_rules/observability.mdc) | All backend source files | Structured logging, tracing, metrics, no silent failures |
| [`security-auditor.mdc`](example_rules/security-auditor.mdc) | Source and Terraform files | OWASP Top 10, injection, SSRF, RBAC, secrets in logs |
| [`legal-policy.mdc`](example_rules/legal-policy.mdc) | Compliance docs | GDPR register checks — no LLM-generated legal copy |

### Architecture
| File | Trigger | Purpose |
|---|---|---|
| [`enterprise-architect.mdc`](example_rules/enterprise-architect.mdc) | `docs/reference/*.md`, source | Module boundary audits, ADR maintenance, cloud-agnostic design |

---

## 18. Example Operating Manual (Prompts)

Use specific, role-based prompts combined with markdown context files.

**1. Task Creation**
> `@task-creator.mdc` Add a task: Integrate Stripe webhooks to update organization subscription statuses. Priority P2.

**2. Planning**
> `@planner.mdc` Analyze `@docs/todo/stripe-webhooks.md` and create an execution plan in `docs/todo/in_progress/task.md`. If anything is ambiguous, ask clarifying questions before writing the plan.

**3. Spec**
> Based on `docs/todo/in_progress/task.md`, write a spec file at `docs/todo/in_progress/task-spec.md`. Include: API contract, expected behavior, edge cases, and a list of test cases that must pass before this is considered done.

**4. Implementation**
> Execute `docs/todo/in_progress/task.md` against `docs/todo/in_progress/task-spec.md`. Do not expand the scope. Run tests after each logical change and self-correct before reporting done.

**5. Context Delegation**
> `@context-delegate.mdc` Gather context for this question: how should we handle idempotent retries for background jobs? Write the summary to `docs/tmp/retry-context.md`.

**6. QA / Testing**
> `@qa-engineer.mdc` Write integration tests for `PaymentService`. Use a real DB container. Ensure tests are deterministic and cover the refund edge case.

**7. Incident Response**
> `@bug-hunter.mdc` Here is a production log: [LOG]. Find the root cause, explain it in one sentence, and apply the most minimal fix possible without refactoring.

**8. Security Audit**
> `@security-auditor.mdc` Review the changes in the last PR for injection vectors, missing auth checks, and secrets in logs.

**9. API Review**
> `@api-design.mdc` Review the new endpoints in `src/routes/orders.ts` for REST naming, error format, and missing pagination.

---

## 19. Team Governance & Metrics

AI adoption must be a team process, not individual magic. If code output increases but review time, defect rates, and rollback rates also increase, AI adoption has not improved delivery — it has merely shifted the cost from writing to debugging.

**Track these metrics:**
* Cycle time per task.
* PR review time.
* Defect rate after merge / Escaped bugs.
* Flaky tests introduced.
* Number of AI iterations before success (measure loop waste).
* Model benchmarks per task category (updated quarterly).

---

## 20. Agentic Engineering is a Living Process

There is no universal playbook. Every project has different constraints, a different codebase, different team habits, and a different risk profile. The model landscape changes every quarter.

**What this means in practice:**
* Revisit your rules every 1-2 months. What was a good rule for GPT-4 may over-constrain Claude 3.7 or under-constrain a newer model.
* Run model benchmarks on your actual tasks — not synthetic benchmarks.
* Iterate on context profiles as your codebase grows. A profile that worked for a 50k LOC codebase will need tuning at 500k LOC.
* Share learnings across the team. AI productivity is multiplicative when the team builds shared conventions, not siloed workflows.
* Treat `docs/`, `.cursor/rules/`, and `AGENTS.md` as first-class engineering artifacts — reviewed, versioned, and owned.

> The teams that get the most out of AI are not the ones that found the perfect setup on day one. They are the ones that iterate on their tooling as deliberately as they iterate on their product.

---

## 21. Beyond the IDE: Agents in Production

This playbook secures your **development-time** AI (Cursor, Copilot). But what happens when you move AI into **production runtime**?

When your LLM agent needs autonomous access to internal APIs, databases, or webhooks, prompt rules are no longer enough. You need deterministic state machines, approval gates, and envelope encryption.

👉 **Next Step:** If your team is building internal AI agents and struggling with compliance, infinite loops, or API security, check out our [Production LLM Agent Security Checklist](https://github.com/PlanVault/production-llm-agent-checklist) or [let's schedule a 15-min architecture chat](https://calendly.com/admin-planvault/15min).

---

**License:** The text of this playbook is released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). The `.mdc` rule snippets in the `example_rules/` directory are released under the [MIT License](https://opensource.org/licenses/MIT). You are free to integrate them into your proprietary codebases.

*This playbook was originally developed while building a production AI-assisted engineering workflow. The practices here emerged from real experience ensuring high-velocity AI assistance does not compromise architecture, security, or testing discipline. Feel free to adopt, fork, and adapt it for your team.*
