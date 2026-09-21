# PROJECT_BOOTSTRAP.md

> **Universal AI-Native Project Bootstrap Specification — Version 2.0 (2026-09-17)**
>
> **Design goal:** autonomous, resumable, cross-agent software delivery with minimal unnecessary human interruption.
>
> **Purpose:** Copy this single file into a new or existing coding project, open that folder in a capable AI coding harness, describe the project you want to build, and instruct the agent to read this file completely and bootstrap the repository.
>
> This file is a **one-time bootstrap specification**, not the permanent day-to-day agent instruction file. After bootstrapping, the repository MUST use a concise root `AGENTS.md` as its canonical operational entry point for coding agents.
>
> Version 2.0 adds a formal **Authorization Envelope**, **Autonomy Protocol**, **Project → Milestone → Workstream → Contract → Atomic Unit → Evidence** work model, **soft and hard checkpoints**, **retry budgets**, and a **Continuity Protocol** that lets a fresh agent resume work without depending on the previous conversation.

---

# 0. How to use this file

Create or open a project folder and place this file at its root:

```text
my-project/
└── PROJECT_BOOTSTRAP.md
```

Then give your coding agent a prompt similar to:

```text
We are building: <describe the project, product, users, constraints, and desired outcome>.

Read PROJECT_BOOTSTRAP.md completely before making changes.

Use it as the controlling bootstrap specification for this repository.

Bootstrap the project appropriately for what I described. Make reasonable,
reversible technical decisions yourself. Ask only when ambiguity would create
meaningful product, security, cost, architecture, legal, or destructive risk.

Set up the repository, source structure, documentation, agent instructions,
development environment, tests, quality gates, CI, and delivery foundations
that are appropriate for this project.

Actually run the generated setup and validation commands before declaring the
bootstrap complete.
```

If the repository already contains code, the agent MUST treat it as an **existing-repository migration/adaptation task**, not as permission to overwrite it.

---

# 1. Mission

Your mission as the bootstrap agent is to transform the current workspace into a repository that is:

- easy for humans to understand;
- easy for capable AI coding agents to navigate;
- explicit about product intent;
- explicit about architecture;
- reproducible to set up;
- deterministic to validate;
- safe to modify;
- difficult to accidentally corrupt;
- resistant to documentation drift;
- efficient in agent context usage;
- testable at the appropriate layers;
- ready for continuous integration;
- ready for controlled delivery when applicable;
- maintainable across many coding-agent products without vendor lock-in.

The goal is **not** to create the largest amount of documentation.

The goal is to create the **smallest sufficient control system** in which the correct way to understand, modify, validate, and deliver the project is obvious.

---

# 2. Core philosophy

Follow these principles throughout the bootstrap and all generated project instructions.

## 2.1 One source of truth per concern

Every durable fact MUST have one canonical owner.

Do not duplicate the same rule or requirement across multiple Markdown files.

Prefer:

```text
AGENTS.md
    -> tells agents where the canonical information lives

docs/PRODUCT.md
    -> product intent and acceptance

docs/ARCHITECTURE.md
    -> current technical system

docs/DESIGN.md
    -> current UI/UX system

docs/QUALITY.md
    -> quality strategy and evidence required

tool configuration
    -> executable style/lint/type/build rules

CI configuration
    -> executable merge/release gates
```

Do not create competing truth sources such as:

```text
RULES.md says one thing
AGENTS.md says another
README.md says something older
SYSTEM.md contains a third version
```

If information is duplicated, consolidate it and leave references instead.

---

## 2.2 Markdown is the control plane, not the enforcement layer

If a machine can reliably enforce a rule, enforce it in tooling rather than relying on prose.

Examples:

```text
formatting        -> formatter
style             -> linter
types             -> compiler/type checker
schema            -> schema validation
behavior          -> automated tests
dependency state  -> manifest + lockfile
security checks   -> scanners/static analysis where appropriate
merge gates       -> CI
```

Use Markdown for:

- intent;
- architecture;
- constraints;
- non-obvious invariants;
- safety rules;
- workflow;
- navigation;
- tradeoffs;
- decisions;
- acceptance criteria;
- information that requires judgment.

---

## 2.3 `AGENTS.md` is a router and operating contract

The permanent root `AGENTS.md` MUST be concise.

It should answer:

```text
What is this project?
Where is everything?
How do I set it up?
How do I run it?
How do I validate a change?
What rules are non-negotiable?
What architecture invariants matter?
What safety boundaries exist?
Where is deeper information documented?
What does "done" mean?
```

It should NOT become an encyclopedia.

Target a root `AGENTS.md` that is typically around **80-160 lines** and preferably remains below roughly **200 lines** unless the repository genuinely requires more.

Move detailed domain-specific information to the document that owns it.

---

## 2.4 Specialized instructions belong close to specialized code

Use nested `AGENTS.md` files only when a subtree genuinely has different:

- build commands;
- architecture rules;
- language conventions;
- testing requirements;
- security boundaries;
- deployment behavior.

Example:

```text
AGENTS.md
apps/
├── web/
│   └── AGENTS.md
└── api/
    └── AGENTS.md
```

Do NOT generate nested instruction files merely because directories exist.

Create them only when they reduce ambiguity.

---

## 2.5 Prefer executable evidence over agent confidence

Never treat phrases such as:

```text
"should work"
"looks correct"
"probably passes"
```

as verification.

Use these explicit verification states:

- **VERIFIED** — the command/check was run and passed.
- **FAILED** — it was run and failed.
- **NOT RUN** — it could have been run but was not.
- **NOT AVAILABLE** — the environment or dependency required to run it is unavailable.

Generated `AGENTS.md` MUST tell future agents not to claim completion without reporting actual verification evidence.

---

## 2.6 Preserve user agency on irreversible decisions

Work autonomously on reversible technical details.

Stop and ask when guessing could create substantial consequences.

Examples requiring confirmation are defined later in this file.

---

# 3. Instruction precedence

When multiple instructions exist, use this precedence order:

1. explicit current user request;
2. repository-local authoritative instructions closest to the affected code;
3. root `AGENTS.md`;
4. canonical project documentation;
5. established repository conventions;
6. ecosystem best practices;
7. your own general preference.

Never silently override a higher-authority instruction with a lower-authority preference.

If two authoritative project documents conflict:

1. identify the conflict;
2. determine which document owns the concern;
3. prefer the canonical owner;
4. update stale references if safe;
5. ask the user only when the conflict represents a real product or architecture choice.

---

# 4. Bootstrap operating mode

You are not merely writing files.

You are establishing an **engineering operating system** for the repository.

The bootstrap process MUST follow this lifecycle:

```text
1. Inspect
2. Understand
3. Classify
4. Contract
5. Decide
6. Scaffold
7. Implement foundation
8. Verify
9. Review
10. Document
11. Generate agent kernel
12. Handoff
```

Each phase is defined below.

---

# 5. Phase 1 — Inspect the workspace before changing it

Before modifying anything, inspect the repository.

Determine:

- whether Git is initialized;
- whether files already exist;
- whether this is greenfield or existing;
- languages present;
- manifests present;
- lockfiles present;
- frameworks present;
- source directories;
- test directories;
- CI configuration;
- Docker/container configuration;
- environment files;
- existing documentation;
- existing agent-instruction files;
- deployment configuration;
- database/migration tooling;
- generated artifacts;
- IDE-specific configuration;
- package manager;
- task runner;
- monorepo tooling;
- license if present.

Look for common files including, but not limited to:

```text
package.json
pnpm-lock.yaml
yarn.lock
package-lock.json
bun.lock

pyproject.toml
poetry.lock
uv.lock
requirements.txt

Cargo.toml
Cargo.lock

go.mod
go.sum

pom.xml
build.gradle
build.gradle.kts

*.sln
*.csproj

composer.json
composer.lock

Gemfile
Gemfile.lock

Makefile
justfile
Taskfile.yml

Dockerfile
compose.yaml
docker-compose.yml

.github/workflows/
.gitlab-ci.yml
azure-pipelines.yml

AGENTS.md
CLAUDE.md
GEMINI.md
.cursor/
.clinerules/
.windsurf/
```

Do not assume the user's preferred stack before inspecting the repository and project description.

---

# 6. Phase 2 — Understand the requested product

Extract the product contract from the user's description.

Identify:

- who the users are;
- what problem is being solved;
- primary workflows;
- required capabilities;
- non-goals;
- constraints;
- expected environment;
- data sensitivity;
- external systems;
- deployment expectations;
- performance expectations;
- offline/online requirements;
- platform targets;
- UI expectations;
- expected scale;
- legal/compliance constraints if explicitly relevant.

Do not begin architecture work before you understand the desired outcome well enough to avoid building the wrong system.

---

# 7. Phase 3 — Classify the project

Classify the project so the scaffold is proportional.

Possible dimensions include:

## 7.1 Product type

Examples:

- library;
- CLI;
- desktop app;
- mobile app;
- web frontend;
- web application;
- backend API;
- SaaS;
- internal tool;
- data pipeline;
- ML/AI system;
- embedded system;
- infrastructure code;
- plugin/extension;
- monorepo;
- multi-service platform.

## 7.2 Risk profile

Estimate whether the project is:

- low risk;
- normal application risk;
- security-sensitive;
- privacy-sensitive;
- infrastructure-sensitive;
- financially consequential;
- production-critical.

Do not over-engineer a small utility as if it were a banking platform.

Do not under-engineer authentication, production infrastructure, secrets, personal data, or irreversible operations.

---

# 8. Phase 4 — Ask only necessary questions

Default to action.

Do NOT interrogate the user about every reversible implementation choice.

Make reasonable decisions yourself when:

- alternatives are low-risk;
- migration is cheap;
- the choice is implementation-level;
- ecosystem convention strongly favors one path;
- the user has delegated technical decisions.

Ask the user when the answer materially changes product behavior, risk, cost, architecture, ownership, or external side effects.

Typical ask-user gates:

## 8.1 Product ambiguity

Ask when:

- two plausible interpretations produce materially different products;
- required acceptance criteria cannot be inferred;
- target users or critical workflow is unclear.

## 8.2 Major architecture choice

Ask when a choice is difficult or expensive to reverse, such as:

- local-only vs cloud-first;
- single-tenant vs multi-tenant;
- relational vs non-relational persistence when domain implications are significant;
- centralized vs distributed architecture when operational consequences are substantial;
- choosing a paid external platform that creates lock-in.

## 8.3 Security/privacy

Ask before:

- weakening authentication;
- reducing authorization;
- storing sensitive data in a new location;
- introducing third-party data sharing;
- bypassing encryption/security controls;
- exposing previously private services publicly.

## 8.4 Destructive operations

Ask before:

- deleting unknown untracked user-authored files;
- rewriting Git history;
- force-pushing;
- dropping real/shared databases;
- destructive production migrations;
- deleting production infrastructure;
- deleting shared cloud resources.

## 8.5 External side effects

Ask before:

- production deployment;
- package publication;
- app-store submission;
- sending real email/SMS;
- charging money;
- provisioning paid cloud resources;
- changing DNS;
- credential rotation;
- modifying shared production systems.

Do NOT ask for permission to create ordinary source files, tests, documentation, local configs, or reversible scaffolding inside the working repository.

---

# AUTONOMY AND CONTINUITY MODEL — Version 2.0

The following rules are fundamental to this operating system.

The repository is the durable system of record.

An individual agent session is **not** a unit of work. It is temporary compute attached to durable project state.

The project must remain understandable and resumable when:

- the context window is exhausted;
- the coding harness restarts;
- the current agent crashes;
- the model changes;
- a different vendor's agent takes over;
- work resumes days or weeks later.

The executing agent MUST optimize for completion of the authorized objective, not for reaching a conversational stopping point.

---

## A. Authorization Envelope

When a user requests implementation, infer an **Authorization Envelope** from the request.

The Authorization Envelope is the set of work that can be performed autonomously without repeatedly asking for permission.

Unless the user narrows the scope, ordinary authorization includes reasonable, reversible, repository-local work necessary to achieve the requested objective, such as:

- inspecting relevant code and documentation;
- creating and editing source files;
- creating tests;
- running tests;
- fixing failures caused by the work;
- refactoring when necessary for correctness;
- updating canonical documentation;
- adding appropriate developer tooling;
- updating local CI configuration;
- updating non-secret local configuration templates;
- creating work contracts and checkpoints;
- making low-risk implementation choices;
- moving automatically from one authorized task/contract to the next.

The agent MUST NOT require routine acknowledgements such as:

```text
continue
proceed
looks good
carry on
yes, do the next part
```

between ordinary work items.

A completed task is normally a reason to select the next authorized task, not a reason to stop.

---

## B. Default autonomy level

Default project autonomy:

```text
HIGH
```

Conceptual levels:

```text
LOW
    Ask before substantial implementation decisions.

NORMAL
    Work autonomously inside the active contract.

HIGH
    Work autonomously across atomic units, contracts, workstreams,
    and ordinary milestone/phase boundaries while remaining inside
    the Authorization Envelope.

UNATTENDED
    Continue through an authorized backlog until blocked, including
    automatic context/session continuation when the harness supports it.
    This level requires an appropriately sandboxed execution environment
    and stronger external-action controls.
```

Unless the user requests otherwise, generated `AGENTS.md` MUST encode **HIGH autonomy**.

---

## C. Human interaction is exception-driven

Human interaction should be reserved for genuine **hard checkpoints**.

Do not turn progress reporting into an approval request.

Good:

```text
Authentication persistence is verified. I am continuing with authorization.
No action is needed from you.
```

Bad:

```text
Authentication persistence is complete. Would you like me to continue?
```

---

## D. Soft checkpoints versus hard checkpoints

### Soft checkpoint

A soft checkpoint persists project state but does NOT require the agent to stop.

Create a soft checkpoint after meaningful transitions such as:

- a contract is completed;
- an architectural decision is made;
- a major test state changes;
- a large refactor becomes coherent;
- a migration is introduced;
- a public interface changes;
- a meaningful blocker is discovered;
- before a risky but authorized local operation;
- when context pressure becomes noticeable.

At a soft checkpoint:

1. validate the current atomic unit as appropriate;
2. update durable project state;
3. update continuity information;
4. optionally create a logical checkpoint commit if repository policy permits;
5. continue automatically.

### Hard checkpoint

A hard checkpoint pauses autonomous execution and requires user input or authorization.

Hard checkpoints include:

- materially ambiguous product requirements that lead to different products;
- major architecture choices that are expensive to reverse and cannot be inferred;
- scope expansion beyond the Authorization Envelope;
- destructive shared/production operations;
- Git history rewriting or force-pushing;
- production deployment when not already explicitly authorized;
- publication to external registries/app stores when not already authorized;
- creation of meaningful paid resources;
- changing DNS;
- credential rotation;
- access to unavailable credentials/accounts;
- weakening security/privacy controls;
- repeated materially different recovery attempts failing;
- irreconcilable requirement contradictions;
- legal/licensing uncertainty with material consequence.

If a hard checkpoint is not reached, continue.

---

## E. Retry budget

A failed command or implementation attempt is NOT automatically a human checkpoint.

For an ordinary blocker:

1. gather evidence;
2. diagnose the likely cause;
3. attempt a reasonable fix;
4. retest;
5. if needed, attempt a materially different approach;
6. retest;
7. continue while new evidence supports further reasonable attempts.

Escalate when:

- multiple materially different attempts have failed;
- further attempts would be blind repetition;
- solving the problem requires information or authority only the user can provide;
- further action would cross a hard checkpoint.

Do not retry the same failed approach indefinitely.

---

# WORK DECOMPOSITION MODEL

Use the following hierarchy for substantial projects:

```text
PROJECT
   │
   └── MILESTONE / OUTCOME
          │
          ├── WORKSTREAM
          │      │
          │      └── CONTRACT
          │             │
          │             ├── ATOMIC UNIT
          │             ├── ATOMIC UNIT
          │             └── ATOMIC UNIT
          │
          └── VERIFICATION EVIDENCE
```

Definitions:

## Project

The durable product or system being built.

Canonical intent lives in `docs/PRODUCT.md`.

## Milestone / Outcome

A coherent product outcome, for example:

```text
M1 — Functional MVP
M2 — Production readiness
M3 — Public release
```

Milestones describe meaningful outcomes, not arbitrary calendar slices.

Reaching an ordinary milestone boundary does NOT require user confirmation unless the next milestone crosses a hard checkpoint or the user explicitly requested review.

## Workstream

A logical area of parallel or related work, such as:

```text
Core engine
API
Frontend
Infrastructure
Quality
Migration
Security
```

A workstream is an organizational concept. Do not create a Markdown file merely because a workstream exists.

## Contract

The primary durable unit of substantial engineering work.

A contract defines:

- outcome;
- scope;
- out of scope;
- acceptance criteria;
- dependencies;
- plan;
- validation;
- risks/decisions;
- handoff state.

Use `docs/work/<contract>.md` for substantial contracts.

Small fixes may use the user's request/conversation as the contract and do not require a file.

## Atomic Unit

The smallest coherent implementation step worth completing and validating before moving on.

Atomic units are normally ephemeral agent tasks, NOT permanent files.

An atomic unit should leave the repository in a reasonably understandable state whenever practical.

## Evidence

Proof that a contract or milestone satisfies its acceptance criteria.

Examples:

- passing tests;
- build artifacts;
- type/lint results;
- integration validation;
- screenshots or UI automation results;
- benchmark results;
- migration validation;
- schema checks.

---

# CONTINUITY PROTOCOL

The repository MUST support cold handoff to a fresh capable agent.

A successor agent should not require the prior conversation to understand:

- the current objective;
- the active contract;
- the current implementation state;
- known failures;
- last verified state;
- exact next action.

Create and maintain:

```text
docs/CONTINUITY.md
```

This file is a concise **generated operational checkpoint**.

It is NOT authoritative for product or architecture facts.

Canonical information remains in its owning document.

---

## Continuity authority

Use this authority model:

```text
CANONICAL / NORMATIVE
    AGENTS.md                 operational rules/router
    docs/PRODUCT.md           product intent
    docs/ARCHITECTURE.md      current architecture
    docs/DESIGN.md            UI/UX rules if present
    docs/QUALITY.md           quality strategy
    docs/SECURITY.md          security model if present
    docs/DELIVERY.md          delivery model if present
    docs/adr/                 significant historical decisions

OPERATIONAL / DURABLE
    docs/STATUS.md            current project snapshot
    docs/work/*.md            active substantial contracts
    .project/WORK.json        optional machine-readable work ledger

GENERATED RESUME SNAPSHOT
    docs/CONTINUITY.md

EPHEMERAL / LOCAL
    .agent/
```

If `CONTINUITY.md` conflicts with a canonical document, the canonical owner wins.

---

## Required `docs/CONTINUITY.md` contents

Keep it short.

It should contain:

```md
# Continuity

> Generated operational checkpoint.
> Canonical requirements and architecture live in their owning documents.

## Active objective

## Active milestone

## Active workstream

## Active contract

## Current position

## Last known good state

## Current verification

## Work in progress

## Blocker / investigation

## Exact next action

## Decisions made since last checkpoint

## Workspace / Git state

## Last checkpoint
```

Do not turn this file into a chronological diary.

Rewrite it to reflect the current resume point.

---

## Known-good checkpoint

Continuity state SHOULD identify the most recent known-good state when available.

Example:

```text
Last known good revision: 4ea81d7

VERIFIED at that state:
- check
- unit tests
- build

Changes after that revision:
- authorization integration in progress
- integration test currently failing
```

This gives a successor a safe reference point without requiring destructive rollback.

---

## Cold-start / successor-agent protocol

A fresh agent entering an existing bootstrapped repository MUST:

```text
1. Read root AGENTS.md.
2. Read docs/CONTINUITY.md.
3. Read docs/STATUS.md.
4. Read the active contract if one exists.
5. Inspect git status.
6. Inspect relevant diff.
7. Inspect recent git history when useful.
8. Read canonical docs relevant to the active work.
9. Run the cheapest useful health/verification check.
10. Compare actual repository state with the handoff.
11. Repair stale continuity metadata if needed.
12. Resume the exact next unblocked action.
```

Do not blindly trust the predecessor's prose.

Use code, Git state, tests, and executable checks as evidence.

---

## Context-pressure protocol

Do NOT intentionally run the context window to zero.

When context pressure is detected:

1. finish the current atomic operation if safe;
2. avoid starting a broad new operation;
3. run the narrowest relevant verification;
4. update the active work contract;
5. update `docs/STATUS.md` if project state materially changed;
6. rewrite `docs/CONTINUITY.md`;
7. record last known good state;
8. preserve exact next action;
9. checkpoint with Git if permitted;
10. use harness compaction/reset/session continuation if available;
11. continue automatically after reset when supported.

If the harness terminates abruptly, prior soft checkpoints should minimize lost context.

---

## Session independence principle

Never structure work so that critical project knowledge exists only in chat.

An agent session may span multiple contracts.

A single contract may span multiple agent sessions.

Neither relationship should affect project correctness.

```text
Agent session 1 ─┐
Agent session 2 ─┼── Contract WORK-014
Agent session 3 ─┘
```

The work belongs to the repository, not the session.

---

# OPTIONAL MACHINE-READABLE WORK LEDGER

For substantial or long-running projects with multiple contracts, create:

```text
.project/WORK.json
```

This file is tracked project-operational metadata.

Do NOT place it in `.agent/`.

Do not create it for tiny projects where it adds no value.

Suggested schema:

```json
{
  "schema_version": 1,
  "project": "example",
  "active_milestone": "M1",
  "active_contract": "WORK-002",
  "contracts": [
    {
      "id": "WORK-001",
      "title": "Domain model",
      "workstream": "Core",
      "status": "verified",
      "depends_on": []
    },
    {
      "id": "WORK-002",
      "title": "Authentication",
      "workstream": "Platform",
      "status": "in_progress",
      "depends_on": ["WORK-001"]
    }
  ]
}
```

Recommended statuses:

```text
candidate
ready
in_progress
blocked
implemented
verifying
verified
deferred
cancelled
```

Rules:

- IDs should be stable.
- Dependencies should reference IDs.
- The ledger tracks execution state, not full requirements.
- Contract Markdown remains the rich explanation.
- `PRODUCT.md` remains the product authority.
- `CONTINUITY.md` remains the current resume packet.
- An agent MUST NOT mark a contract `verified` without evidence.

---

# AUTOMATIC CONTINUATION DIRECTIVE

Generated `AGENTS.md` MUST include a strong version of the following:

```md
## Autonomous continuation

Treat an implementation request as authorization to carry the intended
objective through to completion within the stated scope.

Do not stop merely because:
- one task completed;
- one atomic unit completed;
- one contract completed;
- a test passed;
- files were created;
- a subcomponent works;
- an ordinary milestone/phase boundary was reached;
- you discovered an optional improvement;
- you have a progress update to report.

After completing a unit of work:
1. verify it;
2. persist meaningful state;
3. choose the next highest-priority unblocked work item inside the
   Authorization Envelope;
4. continue automatically.

Human acknowledgement such as "continue", "looks good", "proceed", or
"carry on" is not required between ordinary work items.

Stop only for a hard checkpoint, genuine blocker requiring external
information/authority, completion of the authorized objective, or explicit
user instruction to stop.
```

The objective is engineering completion, not ending the conversational turn.

---

# CHECKPOINT COMMIT POLICY

Git is part of project recovery and continuity.

If the repository/user/harness permits agents to commit automatically:

- create logical commits at verified, coherent checkpoints;
- prefer commits aligned with completed contracts or meaningful atomic groups;
- use meaningful commit messages;
- do not commit known secrets;
- do not commit broken intermediate state merely to "save progress" unless the workflow explicitly uses WIP commits.

If automatic commits are not permitted:

- preserve the worktree;
- maintain accurate `CONTINUITY.md`;
- record uncommitted state explicitly;
- never discard unrelated changes.

Git history is evidence and recovery infrastructure, not a substitute for canonical documentation.

---

# 9. Phase 5 — Choose or preserve the stack

## 9.1 Existing repository

If the repository already has an established stack:

- preserve it unless there is a compelling reason not to;
- do not perform unsolicited framework migrations;
- preserve package manager and lockfile;
- preserve existing build conventions unless broken;
- integrate with existing CI rather than replacing it unnecessarily.

## 9.2 Greenfield repository

For a greenfield project, select a stack based on:

- project requirements;
- maturity;
- ecosystem support;
- maintainability;
- testing quality;
- security;
- deployment target;
- developer ergonomics;
- long-term support;
- availability of deterministic tooling.

Prefer mainstream, well-supported technologies over novelty unless the user requests experimentation.

Avoid unnecessary dependencies.

Avoid introducing infrastructure that the project does not need.

---

# 10. Required repository control plane

The following files form the recommended project documentation/control system.

## 10.1 Always create or establish

```text
AGENTS.md
README.md

docs/
├── PRODUCT.md
├── ARCHITECTURE.md
├── QUALITY.md
├── STATUS.md
├── CONTINUITY.md
└── IDEAS.md
```

## 10.2 Always create operational continuity state

```text
docs/CONTINUITY.md
```

This is a concise generated resume packet for successor agents. It is not a canonical requirements or architecture source.

## 10.3 Create conditionally

```text
docs/DESIGN.md
```

Create when the project has meaningful:

- GUI;
- web UI;
- mobile UI;
- visual design system;
- interaction design;
- user-facing interface conventions.

Do not create it for a pure backend/library unless UI or user interaction genuinely warrants it.

```text
docs/SECURITY.md
```

Create when the project meaningfully involves:

- authentication;
- authorization;
- user data;
- secrets;
- network services;
- public APIs;
- multi-tenancy;
- payment;
- production infrastructure;
- sensitive operations.

```text
docs/DELIVERY.md
```

Create when software is:

- deployed;
- packaged;
- distributed;
- published;
- released;
- installed in production environments.

```text
docs/adr/
```

Create when architectural decisions are significant enough to preserve historically.

```text
docs/work/
```

Create only when substantial tasks benefit from durable work contracts.

```text
.project/WORK.json
```

Create for substantial or long-running projects with multiple contracts, dependencies, or agent handoffs. It is a machine-readable execution ledger, not the requirements authority.

## 10.4 Do NOT create by default

Do not create separate persistent files named:

```text
RULES.md
SYSTEM.md
PROGRESS.md
SUGGESTIONS.md
```

Their useful roles are replaced by clearer ownership:

```text
RULES.md
    -> AGENTS.md + executable tooling + canonical domain docs

SYSTEM.md
    -> docs/ARCHITECTURE.md

PROGRESS.md
    -> docs/STATUS.md

SUGGESTIONS.md
    -> docs/IDEAS.md
```

---

# 11. Canonical ownership model

Use the following ownership rules.

## `AGENTS.md`

Owns:

- agent operating instructions;
- repository map;
- standard commands;
- non-obvious engineering invariants;
- safety boundaries;
- documentation routing;
- definition of done.

Does NOT own detailed product requirements or full architecture prose.

---

## `README.md`

Owns:

- human-first project overview;
- basic setup;
- basic run instructions;
- links to deeper docs.

Avoid turning it into a duplicate of all documentation.

---

## `docs/PRODUCT.md`

Owns:

- problem;
- target users;
- goals;
- non-goals;
- capabilities;
- functional requirements;
- non-functional requirements;
- acceptance criteria;
- domain terminology;
- assumptions;
- open product questions.

This is the canonical **what and why**.

---

## `docs/ARCHITECTURE.md`

Owns the **current** technical system.

Include as relevant:

- system context;
- components;
- boundaries;
- dependencies;
- data flows;
- persistence;
- APIs;
- background jobs;
- integrations;
- deployment topology;
- runtime model;
- failure handling;
- observability;
- architectural invariants;
- known technical debt;
- links to ADRs.

Do not use it as a chronological diary.

---

## `docs/QUALITY.md`

Owns:

- test strategy;
- quality layers;
- minimum validation;
- required CI gates;
- coverage philosophy if relevant;
- test data strategy;
- mocking policy;
- integration boundaries;
- performance checks;
- security checks;
- accessibility checks;
- release validation.

This document answers:

> What evidence is enough to consider a change correct?

---

## `docs/STATUS.md`

Owns the **present state**, not historical narration.

Suggested structure:

```md
# Project status

## Current phase

## Done

## In progress

## Blocked

## Next

## Known issues

## Last verified
- Date:
- Revision:
- Verification:
```

Keep it short.

Do not append an endless chronological progress diary.

Git history, issues, pull requests, releases, and ADRs preserve history better.

---


## `docs/CONTINUITY.md`

Owns the current **resume point** for a successor agent.

It is generated operational state, not product truth.

It answers:

- what objective is active;
- what contract is active;
- what has just been completed;
- what is currently failing;
- what the last known-good state is;
- what exact action should happen next;
- whether uncommitted work exists.

Rewrite it at meaningful soft checkpoints.

Do not append an endless history.

---

## `.project/WORK.json`

When present, owns machine-readable execution status for milestones/contracts and dependencies.

It does NOT own requirements, architecture, or rich contract rationale.

Use it to make unfinished work explicit and successor-agent selection deterministic.

---

## `docs/IDEAS.md`

Owns unapproved future opportunities.

Ideas are NOT requirements.

Each meaningful idea should include:

```text
Title
Problem/opportunity
Expected value
Effort: S / M / L
Risk/tradeoff
Dependencies
Status: candidate / accepted / deferred / rejected
Source/date
```

Agents MAY propose ideas.

Agents MUST NOT silently convert an idea into committed scope.

---

## `docs/DESIGN.md`

When applicable, owns:

- experience principles;
- information architecture;
- responsive strategy;
- design tokens;
- typography;
- spacing;
- color semantics;
- components;
- state behavior;
- loading states;
- empty states;
- error states;
- success states;
- forms;
- validation;
- keyboard interactions;
- accessibility;
- motion;
- content/voice;
- reference screens.

Do not duplicate CSS/token files verbatim; reference canonical implementation where appropriate.

---

## `docs/SECURITY.md`

When applicable, owns:

- trust boundaries;
- threat assumptions;
- authentication model;
- authorization model;
- sensitive data;
- secrets handling;
- security-sensitive workflows;
- logging restrictions;
- dependency/security update strategy;
- security validation;
- incident-relevant considerations.

Never place actual secrets in this document.

---

## `docs/DELIVERY.md`

When applicable, owns:

- environments;
- build artifacts;
- packaging;
- release process;
- deployment process;
- configuration model;
- migrations;
- rollback;
- observability expectations;
- smoke tests;
- release verification.

---

## `docs/adr/*.md`

Own historical reasoning for significant architectural decisions.

Use ADRs only for decisions that are:

- architecturally significant;
- expensive to reverse;
- likely to be questioned later;
- important for future agents to understand.

Do not create an ADR for every small implementation decision.

Suggested ADR structure:

```md
# ADR-XXXX: Decision title

- Status: proposed | accepted | superseded | rejected
- Date:
- Supersedes:
- Superseded by:

## Context

## Decision

## Alternatives considered

## Consequences

## Validation / follow-up
```

Accepted ADRs should normally remain historical records.

If a decision changes, create a new ADR that supersedes the old one rather than rewriting history.

---

## `docs/work/<task>.md`

Use only for work that is:

- multi-step;
- cross-cutting;
- risky;
- likely to span multiple sessions;
- dependent on explicit acceptance criteria.

Suggested structure:

```md
# Work: <name>

## Outcome

## Scope

## Out of scope

## Acceptance criteria

## Dependencies

## Current state

## Plan / atomic units

## Validation / evidence

## Risks / decisions

## Handoff / exact next action
```

Small fixes do not require work files.

---

# 12. Recommended initial file tree

A typical greenfield repository should converge toward:

```text
project/
│
├── PROJECT_BOOTSTRAP.md
├── AGENTS.md
├── README.md
├── .gitignore
├── .editorconfig                 # when useful
├── .env.example                  # when environment config exists
│
├── docs/
│   ├── PRODUCT.md
│   ├── ARCHITECTURE.md
│   ├── QUALITY.md
│   ├── STATUS.md
│   ├── CONTINUITY.md
│   ├── IDEAS.md
│   ├── DESIGN.md                 # conditional
│   ├── SECURITY.md               # conditional
│   ├── DELIVERY.md               # conditional
│   ├── adr/
│   │   └── README.md             # optional index/instructions
│   └── work/
│       └── ...                   # substantial work only
│
├── .project/                     # conditional; tracked operational metadata
│   └── WORK.json                 # substantial/long-running projects only
│
├── src/                          # or ecosystem-native equivalent
├── tests/                        # or ecosystem-native equivalent
│
├── <manifest>
├── <lockfile>
│
└── <CI configuration>
```

Adapt to ecosystem norms.

Do not force `src/` if the ecosystem/framework has a strong standard layout.

---

# 13. Git and version control

If this is a greenfield repository and Git is available:

- initialize Git unless the environment/user context clearly says not to;
- create an appropriate `.gitignore`;
- do not commit secrets;
- do not commit machine-local caches;
- do commit deterministic configuration;
- do commit lockfiles for applications unless ecosystem conventions strongly differ;
- do not create commits unless the user or harness workflow requests it.

If Git already exists:

- preserve repository history;
- inspect current status before modifying;
- avoid overwriting unrelated local changes;
- do not reset user work;
- do not discard changes you did not create.

---

# 14. Safe deletion and archive policy

A universal "never delete any file" rule is NOT recommended.

It creates stale-code pollution and can confuse future humans and agents.

Use this policy instead.

## 14.1 Tracked obsolete source

If a tracked file is clearly obsolete and deletion is part of the intended change:

- deleting it normally is acceptable;
- Git preserves recoverability.

## 14.2 Generated/cache/build artifacts

If they are safe and reproducible:

- delete freely when appropriate;
- ensure they are ignored when they should not be versioned.

Examples:

```text
dist/
build/
coverage/
.cache/
.tmp/
node_modules/
__pycache__/
```

Exact names depend on stack.

## 14.3 Unknown untracked files

Do NOT destroy unknown untracked files casually.

They may contain user work.

If an unknown untracked file blocks work:

- inspect it;
- preserve it;
- ask if its purpose cannot be safely inferred.

## 14.4 Ambiguous valuable artifacts

If a file must be displaced but may contain user-authored or valuable information, archive it instead of deleting.

Use:

```text
.archive/
├── README.md
└── YYYY-MM-DD/
    └── original/path/file.ext
```

`.archive/README.md` should record:

- original path;
- archive path;
- date;
- reason;
- replacement if any;
- agent/user responsible if known.

Do NOT use `.archive/` as a dumping ground for ordinary obsolete tracked source.

## 14.5 Gitignore decision for `.archive/`

Default:

- keep `.archive/` ignored if it contains temporary local safety copies;
- version it only if the user explicitly wants archival artifacts preserved in repository history.

---

# 15. Local agent workspace state

Do not pollute the repository with ephemeral reasoning logs.

If the harness or workflow benefits from local transient state, use a dedicated ignored directory such as:

```text
.agent/
```

Possible contents:

```text
.agent/
├── scratch/
├── evidence/
├── tmp/
└── local-notes/
```

Default `.gitignore`:

```gitignore
.agent/
```

Do not store authoritative project facts only inside `.agent/`.

`.agent/` is ephemeral/local and should normally be ignored.

By contrast, `.project/` is reserved for small, structured, **tracked operational metadata** that must survive agent/session replacement, such as optional `WORK.json`.

Any durable information discovered during work must be promoted to the correct canonical document, test, configuration, issue, or code.

---

# 16. Cross-agent compatibility

The root `AGENTS.md` is the canonical project instruction file.

Do not maintain multiple full copies for different vendors.

Create tiny adapters only when useful.

## 16.1 Claude Code

If Claude Code compatibility is desired, create:

```text
CLAUDE.md
```

with the minimal supported import/reference approach for `AGENTS.md`.

Prefer a tiny adapter such as:

```md
@AGENTS.md
```

if supported by the installed Claude Code version.

Do not duplicate the full contents of `AGENTS.md`.

## 16.2 Gemini CLI

If Gemini CLI is expected:

- configure it to use `AGENTS.md` if supported; or
- create a minimal `GEMINI.md` that imports/references `AGENTS.md`.

Do not maintain independent instructions.

## 16.3 Cursor / Cline / Windsurf / Copilot / other agents

Use native repository-rule adapters only when they materially improve compatibility.

The adapter must point to or summarize the canonical `AGENTS.md` without becoming a second source of truth.

General rule:

```text
one canonical instruction source
+
minimal adapters
=
portable agent compatibility
```

---

# 17. Command surface

Every repository MUST expose a small, discoverable semantic command surface.

Future agents should not need to guess how to work with the project.

Provide the semantic equivalent of:

```text
setup
dev
check
test
test:e2e       # if applicable
build
ci
```

Not every project needs every command, but the operational meanings should exist.

## 17.1 Command definitions

### setup

Reproducibly prepare the development environment.

Examples:

- install dependencies;
- generate local config templates;
- initialize local development services where safe.

### dev

Start the project for local development.

### check

Run fast static validation.

Examples:

- formatting verification;
- lint;
- types;
- static checks;
- schema checks.

### test

Run the primary automated test suite.

### test:e2e

Run end-to-end tests if the project has meaningful user/system flows.

### build

Produce the intended build/package/artifact.

### ci

Run the complete required pre-merge validation.

The `ci` command should represent the strongest deterministic local approximation of CI.

---

# 18. Prefer ecosystem-native command mechanisms

Do not add a new task runner merely to satisfy abstract naming.

Examples:

## Node / JavaScript / TypeScript

Prefer scripts in:

```text
package.json
```

Example semantic surface:

```json
{
  "scripts": {
    "dev": "...",
    "check": "...",
    "test": "...",
    "test:e2e": "...",
    "build": "...",
    "ci": "..."
  }
}
```

## Python

Prefer the existing modern project configuration and established project scripts/tasks.

Use the repository's chosen package/environment manager consistently.

## Rust

Prefer Cargo-native commands plus minimal scripts only when orchestration is necessary.

## Go

Prefer Go tooling plus minimal scripts/Make targets only when needed.

## .NET / Java / other ecosystems

Use native build systems and conventional commands.

The goal is not universal syntax.

The goal is universal discoverability.

Document exact commands in `AGENTS.md`.

---

# 19. Reproducibility

The bootstrap MUST establish deterministic development where practical.

Prefer:

- declared dependency versions;
- lockfiles;
- explicit runtime version constraints;
- container/devcontainer only where valuable;
- reproducible setup commands;
- `.env.example` for configuration shape;
- no real credentials in repository;
- no undocumented manual setup steps.

If setup requires external systems, document:

- what is required;
- how to obtain it;
- whether a local substitute exists;
- which steps cannot be automated.

---

# 20. Environment configuration

If the application uses environment variables:

Create:

```text
.env.example
```

containing:

- variable names;
- safe placeholder values;
- comments when necessary.

Never include:

- passwords;
- API keys;
- tokens;
- private keys;
- real database credentials;
- cloud credentials;
- production secrets.

Ensure real `.env` files are ignored unless a framework explicitly uses a safely versioned non-secret variant.

---

# 21. Testing strategy

Testing must be proportional to risk and architecture.

Do not mechanically add every type of test.

Consider:

- unit;
- component;
- integration;
- contract/schema;
- database;
- end-to-end;
- visual regression;
- accessibility;
- security;
- performance;
- smoke tests.

`docs/QUALITY.md` MUST explain which layers apply and why.

---

# 22. Test the boundaries that can break

Prioritize tests around:

- business logic;
- parsing;
- calculations;
- state transitions;
- persistence;
- API contracts;
- external adapters;
- authentication;
- authorization;
- destructive operations;
- migrations;
- previously broken behavior;
- critical user journeys.

Avoid tests that only mirror implementation details without protecting behavior.

---

# 23. Regression rule

When fixing a reproducible bug:

1. reproduce the failure when practical;
2. add or modify a test that detects the bug;
3. implement the fix;
4. run the targeted test;
5. run broader relevant validation.

If a regression test is impractical, document why in the handoff.

---

# 24. Quality gates

Define appropriate gates for the stack.

A typical application may require:

```text
format/lint
typecheck/compiler
unit tests
integration tests
build
security checks
e2e/smoke checks
```

The generated repository should make these executable through the command surface.

`AGENTS.md` MUST tell future agents which gate is required before completion.

---

# 25. CI

If the project is expected to live in a hosted Git repository and CI is appropriate, establish CI.

Default to the hosting system already present.

Examples:

- GitHub Actions;
- GitLab CI;
- Azure Pipelines;
- other existing CI.

Do not migrate CI providers without need.

CI should reuse the same underlying commands humans and agents run locally.

Avoid hidden CI-only logic where possible.

A typical CI flow:

```text
checkout
runtime/toolchain setup
dependency install
check
test
build
additional required validation
```

Use caching where safe and straightforward.

Do not optimize CI prematurely at the cost of clarity.

---

# 26. Delivery and deployment

Do not automatically deploy merely because deployment configuration exists.

A build pipeline and a production deployment are different authority levels.

Where applicable, establish:

- deterministic build artifact;
- environment separation;
- migration strategy;
- smoke checks;
- rollback path;
- deployment documentation.

Production delivery SHOULD remain controlled by explicit user/team authorization or existing deterministic release workflows.

---

# 27. Security baseline

For security-relevant applications:

- validate input at trust boundaries;
- use least privilege;
- avoid secret exposure;
- separate authorization from UI visibility;
- use proven cryptographic libraries;
- do not invent authentication protocols;
- do not log secrets;
- avoid leaking sensitive personal data;
- prefer secure defaults;
- document trust boundaries;
- add dependency/security scanning when appropriate.

Never weaken security merely to make tests or local development easier without clearly scoped development-only behavior.

---

# 28. Dependencies

Before adding a dependency, ask:

- does the standard library/current framework already solve this?
- is the dependency maintained?
- is its license acceptable for the project?
- is it security-sensitive?
- does it materially reduce complexity?
- does it create lock-in?
- does it significantly increase bundle/runtime footprint?

Do not add dependencies for trivial functionality.

Use the existing package manager.

Update the lockfile together with manifest changes.

---

# 29. Architecture rules

Prefer:

- clear boundaries;
- explicit dependency direction;
- boring, understandable abstractions;
- small modules with coherent responsibility;
- interfaces at real boundaries;
- observable failures;
- deterministic state transitions.

Avoid:

- speculative abstraction;
- unnecessary microservices;
- framework proliferation;
- premature distributed architecture;
- global mutable state where avoidable;
- cleverness that reduces maintainability.

Choose architecture because it fits the product, not because it is fashionable.

---

# 30. Observability

For deployable/runtime services, consider:

- structured logs;
- request/job correlation where relevant;
- meaningful error reporting;
- health checks;
- metrics;
- tracing if complexity warrants it.

Do not introduce a large observability stack for a tiny local utility.

Document what exists in `ARCHITECTURE.md` and/or `DELIVERY.md`.

---

# 31. UI and design systems

If the project has a user-facing UI:

Create `docs/DESIGN.md`.

At bootstrap, define enough design direction to prevent arbitrary interface drift.

At minimum consider:

- visual principles;
- responsive behavior;
- typography;
- spacing;
- color semantics;
- component reuse;
- accessibility;
- state handling;
- keyboard behavior;
- error presentation;
- loading behavior;
- empty states.

If the user supplied a design system, screenshots, mockups, or design requirements, they take precedence over agent preference.

---

# 32. Accessibility

For user-facing web/mobile/desktop interfaces, accessibility is part of correctness.

As applicable, consider:

- semantic structure;
- labels;
- focus management;
- keyboard navigation;
- color contrast;
- reduced motion;
- screen reader behavior;
- error identification.

Add automated checks when they provide meaningful value, but do not mistake automated accessibility testing for complete accessibility validation.

---

# 33. API contracts and schemas

For systems with APIs or structured messages:

- prefer explicit schemas/contracts;
- validate inputs;
- version externally consumed interfaces deliberately;
- test important contracts;
- avoid silent breaking changes.

Document public contract behavior where users/integrators need it.

---

# 34. Database and migrations

If persistence exists:

- use the ecosystem's standard migration mechanism;
- keep migrations versioned;
- do not mutate production/shared databases during bootstrap;
- document local initialization;
- test migration behavior when risk warrants it;
- distinguish reversible code changes from potentially irreversible data changes.

Production-destructive migrations require explicit authorization.

---

# 35. Generated code

If code or artifacts are generated:

Document:

- source of truth;
- generation command;
- whether output is committed;
- when regeneration is required.

Never manually edit generated artifacts when the real source should be changed.

---

# 36. Documentation freshness rule

Documentation is part of the change when the facts it owns change.

Future agents MUST update the canonical owner when they change:

```text
product capability        -> PRODUCT.md
architecture              -> ARCHITECTURE.md / ADR
design system             -> DESIGN.md
quality strategy          -> QUALITY.md
security model            -> SECURITY.md
delivery process          -> DELIVERY.md
agent command/rule        -> AGENTS.md
project state             -> STATUS.md
future unapproved idea    -> IDEAS.md
```

Do not update unrelated docs for cosmetic completeness.

---

# 37. Avoid documentation duplication

Bad:

```text
README contains exact architecture
ARCHITECTURE repeats it
AGENTS repeats it
CLAUDE repeats it
```

Good:

```text
ARCHITECTURE.md = canonical architecture
README.md       = short human overview + link
AGENTS.md       = agent pointer + critical invariants
CLAUDE.md       = tiny adapter to AGENTS.md
```

---

# 38. Project status discipline

`docs/STATUS.md` should stay compact.

Update it at meaningful milestones, not after every command.

A good `STATUS.md` lets a new human or agent answer in under a minute:

- what phase are we in?
- what is done?
- what is being worked on?
- what is blocked?
- what should happen next?
- what is currently known to be broken?
- when was the project last verified?

---

# 39. Ideas discipline

At the end of substantial work, agents SHOULD consider whether they discovered:

- usability improvements;
- architecture simplifications;
- missing tests;
- performance opportunities;
- automation opportunities;
- design improvements;
- security improvements;
- developer-experience improvements.

If worthwhile and outside current scope, add them to `docs/IDEAS.md`.

Do not derail the current task to implement them without approval when they represent meaningful scope expansion.

---

# 40. Working lifecycle after bootstrap

Generate `AGENTS.md` so future agents follow this operational lifecycle:

```text
LOAD / UNDERSTAND
        ↓
CONTRACT
        ↓
IMPLEMENT ATOMIC UNIT
        ↓
VERIFY
        ↓
REVIEW
        ↓
CHECKPOINT
        ↓
MORE AUTHORIZED WORK?
    ┌─────────────┐
   YES            NO
    │              │
    ▼              ▼
CONTINUE       FINAL HANDOFF
```

`CHECKPOINT` normally means persist state and continue. It does not imply waiting for the user.

---

# 41. UNDERSTAND

Before changing code, future agents should:

- read root `AGENTS.md`;
- read nearest nested `AGENTS.md` if present;
- inspect relevant source;
- inspect relevant tests;
- read the canonical docs that own the affected concern.

Avoid broad repository exploration when precise navigation is possible.

---

# 42. CONTRACT

For a small task, the conversation plus clear acceptance criteria is enough.

For substantial work, use `docs/work/<task>.md`.

Before implementation, clarify:

- desired outcome;
- in scope;
- out of scope;
- acceptance criteria;
- validation needed.

---

# 43. IMPLEMENT

Make the smallest coherent change that satisfies the contract.

Do not mix unrelated refactors unless they are required to safely complete the task.

Avoid "while we are here" rewrites.

---

# 44. VERIFY

Run narrow checks first for fast feedback.

Typical order:

```text
targeted test
↓
affected package/module validation
↓
repository check/test
↓
full ci-equivalent validation when appropriate
```

Record failures honestly.

Never claim a check passed unless it actually ran successfully.

---

# 45. REVIEW

Before handoff, inspect the final diff or changed files.

Look for:

- unrelated edits;
- accidental deletions;
- stale debug code;
- secrets;
- dead code;
- duplicate logic;
- unnecessary complexity;
- missing tests;
- stale docs;
- unexpected generated artifacts;
- backwards-compatibility issues;
- security regressions.

---

# 46. DOCUMENT

Update only the canonical documents whose owned facts changed.

If an architecture decision was significant, add an ADR.

If a new improvement idea was discovered but is outside scope, add it to `IDEAS.md`.

---

# 47. HANDOFF

Every substantial completed task should report:

- what changed;
- what was verified;
- what failed;
- what was not run;
- assumptions;
- remaining risks;
- remaining blockers;
- next highest-value step.

Use explicit verification states.

---

# 48. Definition of done

The bootstrap is not complete merely because files were created.

Before reporting bootstrap completion, verify as much of the following as applies:

- repository structure exists;
- dependencies can be installed;
- project can start or execute;
- static checks run;
- tests run;
- build succeeds;
- CI configuration is syntactically plausible and reuses local commands;
- documentation matches actual commands;
- `.gitignore` excludes secrets/caches;
- `.env.example` is safe;
- no obvious secret was introduced;
- `AGENTS.md` exists and is concise;
- adapters reference the canonical instruction file rather than duplicate it;
- `STATUS.md` reflects reality;
- `CONTINUITY.md` contains an exact resume point;
- a fresh agent could identify the next action without the previous conversation.

If something cannot be verified, report it as `NOT AVAILABLE` or `NOT RUN`.

---

# 49. Bootstrap sequencing for greenfield projects

Use approximately this sequence:

```text
1. inspect workspace
2. parse user product description
3. identify critical unanswered questions
4. ask only if necessary
5. choose stack
6. establish PRODUCT.md
7. establish architecture
8. create skeleton source structure
9. create executable setup/dev/check/test/build/ci surface
10. add minimal working implementation
11. add tests
12. add formatter/linter/type checks as appropriate
13. add CI
14. add conditional DESIGN/SECURITY/DELIVERY docs
15. add README
16. verify commands
17. self-review
18. generate concise AGENTS.md from verified reality
19. generate tiny harness adapters if useful
20. establish/update CONTINUITY.md
21. create/update optional .project/WORK.json when project scale warrants it
22. update STATUS.md
23. handoff or continue automatically if authorized work remains
```

Important:

Generate the final `AGENTS.md` **after** actual commands and structure have been verified, so it documents reality rather than intention.

---

# 50. Bootstrap sequencing for existing repositories

If files/code already exist:

```text
1. inspect without destructive changes
2. identify existing conventions
3. run existing setup/tests if practical
4. locate existing agent/rule files
5. identify documentation conflicts
6. preserve existing stack and command surface
7. add missing control documents incrementally
8. consolidate duplicate rules
9. create/refresh AGENTS.md
10. improve quality gates only where justified
11. preserve user modifications
12. verify no regressions
13. reconstruct/update active contracts and optional WORK.json when useful
14. update CONTINUITY.md with the exact resume point
15. update STATUS.md
16. continue automatically if authorized work remains; otherwise handoff
```

Do NOT:

- wipe the repository;
- replace working tooling merely to match this template;
- rename every directory to your preferred convention;
- replace CI providers without need;
- delete unfamiliar user files;
- convert architecture wholesale unless explicitly requested.

This specification is an operating model, not an excuse for forced migration.

---

# 51. Existing agent-instruction migration

If the repository contains multiple instruction systems such as:

```text
AGENTS.md
CLAUDE.md
GEMINI.md
.cursorrules
.cursor/rules/
.clinerules/
.windsurfrules
.github/copilot-instructions.md
```

Inspect them.

Then:

1. identify unique useful rules;
2. identify duplicates;
3. identify stale/conflicting rules;
4. consolidate durable cross-agent rules into `AGENTS.md`;
5. move domain facts to canonical project docs;
6. move enforceable rules into tooling;
7. keep only small vendor adapters where necessary.

Do not discard unique instructions without preserving their intent.

---

# 52. Template — generated root `AGENTS.md`

Generate a project-specific version of the following.

Do NOT leave placeholders that should have been resolved during bootstrap.

```md
# AGENTS.md

## Project

<1-3 sentences describing the project and its purpose.>

## Repository map

- `<path>` — <purpose>
- `<path>` — <purpose>
- `docs/PRODUCT.md` — product requirements and acceptance criteria
- `docs/ARCHITECTURE.md` — current system architecture
- `docs/QUALITY.md` — quality and test strategy
- `docs/STATUS.md` — current project state
- `docs/IDEAS.md` — unapproved future improvements
- `<conditional docs>` — <purpose>

## Standard commands

- Setup: `<exact command>`
- Dev: `<exact command>`
- Check: `<exact command>`
- Test: `<exact command>`
- E2E: `<exact command or N/A>`
- Build: `<exact command>`
- CI-equivalent: `<exact command>`

Do not invent alternate commands unless these are broken and you are fixing them.

## Engineering invariants

- <only important project-specific invariant>
- <only important project-specific invariant>

## Autonomy

Default autonomy is HIGH.

Continue automatically across ordinary tasks, contracts, workstreams, and
milestone boundaries while work remains inside the user's Authorization
Envelope.

Progress updates do not require acknowledgement.

Stop only at a hard checkpoint, a genuine external blocker, completion of the
authorized objective, or explicit instruction to stop.

## Cold-start / resume

When entering with no prior conversation:
1. Read `docs/CONTINUITY.md`.
2. Read `docs/STATUS.md`.
3. Read the active work contract if present.
4. Inspect Git status and relevant diff/history.
5. Run the cheapest useful health check.
6. Resume the exact next unblocked action.

## Workflow

1. Understand relevant code/tests/docs.
2. Define or confirm acceptance criteria.
3. Make the smallest coherent change.
4. Run targeted validation.
5. Run the required repository gate before completion.
6. Review the diff.
7. Update canonical docs if owned facts changed.
8. Report verification evidence.

## Testing

<project-specific test expectations>

Regression fixes should include a regression test when practical.

Never claim a test/check passed unless it was actually run.

Use verification states: VERIFIED / FAILED / NOT RUN / NOT AVAILABLE.

## Safety

- Never expose or commit secrets.
- Do not discard unrelated user changes.
- Do not delete unknown untracked user files.
- Tracked obsolete files may be removed normally when the change requires it; Git is the recovery mechanism.
- Ask before destructive production/shared operations, history rewrites, force pushes, publishing, paid provisioning, credential rotation, or production deployment.
- <project-specific safety boundaries>

## Documentation ownership

- Product behavior/requirements -> `docs/PRODUCT.md`
- Current architecture -> `docs/ARCHITECTURE.md`
- Significant architecture decisions -> `docs/adr/`
- Test/quality strategy -> `docs/QUALITY.md`
- UI/UX system -> `docs/DESIGN.md` if present
- Security model -> `docs/SECURITY.md` if present
- Delivery/release -> `docs/DELIVERY.md` if present
- Current state -> `docs/STATUS.md`
- Future unapproved ideas -> `docs/IDEAS.md`

Avoid duplicating canonical facts.

## Definition of done

A change is complete when:
- acceptance criteria are satisfied;
- relevant tests/checks pass;
- required CI-equivalent validation has been run when practical;
- the diff has been reviewed;
- relevant canonical docs are current;
- remaining failures or unavailable checks are explicitly reported.
```

Keep the final generated file concise.

---

# 53. Template — `docs/PRODUCT.md`

```md
# Product

## Problem

## Target users

## Goals

## Non-goals

## Primary workflows

## Functional requirements

## Non-functional requirements

## Acceptance criteria

## Domain terminology

## Assumptions

## Open product questions
```

Populate with actual known information.

Do not invent major product requirements without support.

---

# 54. Template — `docs/ARCHITECTURE.md`

```md
# Architecture

## System overview

## Context

## Components

## Boundaries and responsibilities

## Data flow

## Persistence

## External integrations

## Runtime / deployment topology

## Failure handling

## Observability

## Security-relevant architecture

## Architectural invariants

## Known technical debt

## Architecture decisions

See `docs/adr/`.
```

Use only relevant sections.

---

# 55. Template — `docs/QUALITY.md`

```md
# Quality strategy

## Quality goals

## Validation layers

### Static checks

### Unit tests

### Integration tests

### Contract/schema tests

### End-to-end tests

### Accessibility

### Security

### Performance

## Standard quality commands

## Test data and fixtures

## Mocking / fakes policy

## Required pre-merge gate

## Known gaps
```

Remove irrelevant sections rather than filling them with noise.

---

# 56. Template — `docs/STATUS.md`

```md
# Project status

## Current phase

## Done

## In progress

## Blocked

## Next

## Known issues

## Last verified

- Date:
- Revision:
- Verification:
```

Keep this a current snapshot.

---

# 57. Template — `docs/IDEAS.md`

```md
# Ideas

This file contains unapproved future improvements.

Items here are not committed scope.

## Candidate ideas

### <Title>

- Status: candidate
- Expected value:
- Effort: S / M / L
- Risk / tradeoff:
- Dependencies:
- Source / date:

#### Problem or opportunity

#### Proposed direction
```

Use statuses:

```text
candidate
accepted
deferred
rejected
```

Move accepted work into the relevant product/work contract rather than treating `IDEAS.md` itself as the implementation plan.

---

# 58. Template — `docs/DESIGN.md`

Create only if relevant.

```md
# Design system

## Experience principles

## Information architecture

## Layout and responsive behavior

## Design tokens

## Typography

## Color semantics

## Spacing

## Components

## Interaction patterns

## Loading states

## Empty states

## Error states

## Success states

## Forms and validation

## Keyboard interaction

## Accessibility

## Motion

## Content and voice

## Reference screens / flows
```

---

# 59. Template — `docs/SECURITY.md`

Create only if relevant.

```md
# Security model

## Security goals

## Trust boundaries

## Authentication

## Authorization

## Sensitive data

## Secrets

## Input and output boundaries

## Logging restrictions

## External integrations

## Dependency security

## Security validation

## Known risks

## Incident / recovery considerations
```

---

# 60. Template — `docs/DELIVERY.md`

Create only if relevant.

```md
# Delivery

## Environments

## Configuration

## Build artifact

## Release process

## Deployment process

## Database migrations

## Smoke validation

## Rollback

## Observability after release

## Recovery / troubleshooting
```

---

# 61. Template — ADR index

If `docs/adr/` is created, optionally create:

```md
# Architecture Decision Records

Use ADRs for significant, difficult-to-reverse, or historically important
technical decisions.

Do not create ADRs for routine implementation details.

Accepted ADRs are historical records. If a decision changes, add a new ADR
that supersedes the old one rather than rewriting accepted history.
```

---

# Template — `docs/CONTINUITY.md`

```md
# Continuity

> Generated operational checkpoint. Canonical product, architecture, design,
> quality, security, and delivery facts live in their owning documents.

## Active objective

## Active milestone

## Active workstream

## Active contract

## Current position

## Last known good state

## Current verification

- VERIFIED:
- FAILED:
- NOT RUN:
- NOT AVAILABLE:

## Work in progress

## Blocker / investigation

## Exact next action

## Decisions made since last checkpoint

## Workspace / Git state

## Last checkpoint
```

Keep it concise enough for a cold agent to read immediately.

---

# Template — optional `.project/WORK.json`

```json
{
  "schema_version": 1,
  "project": "<project>",
  "active_milestone": "<id-or-null>",
  "active_contract": "<id-or-null>",
  "contracts": [
    {
      "id": "WORK-001",
      "title": "<title>",
      "workstream": "<workstream>",
      "status": "ready",
      "depends_on": [],
      "contract_file": "docs/work/WORK-001-<slug>.md"
    }
  ]
}
```

Only add fields that improve deterministic continuation.

Avoid turning the ledger into a second requirements database.

---

# Successor-agent startup contract

A generated root `AGENTS.md` MUST tell a cold successor to:

```text
read AGENTS.md
→ read CONTINUITY.md
→ read STATUS.md
→ read active contract
→ inspect Git status/diff/history
→ run a cheap health check
→ compare evidence with handoff
→ continue automatically
```

If no active contract exists, select the highest-priority unblocked authorized contract from `WORK.json` when present, otherwise derive it from `STATUS.md` and project requirements.

---

# 62. Source-code style

Prefer style consistency with the existing codebase.

For greenfield projects:

- use the ecosystem-standard formatter;
- use an established linter/static analyzer where useful;
- avoid custom style rules unless needed;
- let tooling enforce formatting.

Do not place long style guides in `AGENTS.md`.

---

# 63. Comments and documentation in code

Write comments that explain:

- why;
- invariants;
- non-obvious tradeoffs;
- constraints;
- externally surprising behavior.

Do not comment obvious syntax.

Prefer clear names and structure over excessive commentary.

---

# 64. Error handling

Do not swallow errors silently.

At system boundaries:

- validate;
- return actionable errors;
- log appropriately;
- avoid leaking sensitive information.

Use the ecosystem's idiomatic error mechanisms.

---

# 65. Feature flags

Use feature flags only when they solve a real release/migration need.

Do not create permanent flag debt.

Document ownership and removal conditions for meaningful flags.

---

# 66. Performance

Do not optimize without evidence unless the domain has clear performance constraints.

Where performance matters:

- define target;
- measure baseline;
- optimize;
- measure again;
- preserve benchmark or regression test when valuable.

---

# 67. Backwards compatibility

If the project already has users, public APIs, data formats, or persisted data:

- treat compatibility as a requirement unless the user explicitly approves a break;
- identify breaking changes;
- provide migration paths where appropriate.

---

# 68. Licensing and third-party assets

Respect existing project licensing.

Before introducing third-party code/assets with meaningful licensing implications:

- identify license;
- ensure compatibility;
- preserve required attribution.

Do not copy proprietary code from unknown sources.

---

# 69. Secret scanning before handoff

Before bootstrap completion and before major handoffs, inspect changed files for obvious accidental secrets.

Examples:

```text
API keys
access tokens
private keys
passwords
cloud credentials
database credentials
```

If a real credential appears to have been exposed, do not merely remove it from the latest file.

Warn that rotation may be necessary because repository/history/log exposure may already have occurred.

---

# 70. CI as deterministic authority

Where CI exists, it is the authoritative machine gate for merge/release checks.

Agents should run the local equivalent before handoff when practical.

Do not create a situation where:

```text
local validation != CI validation
```

without documenting why.

---

# 71. Failure policy

If setup or tests fail during bootstrap:

Do not hide the failure.

Attempt reasonable diagnosis.

Fix bootstrap-created failures when possible.

If the failure depends on unavailable external services or credentials:

- do not invent credentials;
- document the requirement;
- mark validation `NOT AVAILABLE`;
- still validate all independent layers that can run.

---

# 72. No fake completeness

Do not fill documents with fabricated specifics just to make them look complete.

Use:

```text
TBD
Open question
Not yet applicable
```

sparingly and only when the unknown is real.

Prefer resolving reversible technical details yourself.

Ask the user when the unknown is consequential.

---

# 73. No scaffolding theater

Do not add:

- Kubernetes;
- Terraform;
- microservices;
- event buses;
- service meshes;
- complex observability;
- elaborate release tooling;
- dozens of Markdown files;

unless the project actually needs them.

Optimization means minimizing unnecessary system complexity.

---

# 74. No context theater

Do not assume adding more agent instructions improves coding quality.

Permanent agent context should contain high-value information.

When an agent makes a repeatable mistake, decide the best prevention mechanism:

```text
Can a formatter prevent it?
Can a linter prevent it?
Can a type/schema prevent it?
Can a test catch it?
Can CI block it?
Is it relevant only to one subtree?
Is it a procedural script?
Is it genuinely worth permanent agent context?
```

Only add to `AGENTS.md` when persistent context is the right solution.

---

# 75. Refactoring policy

Refactor when:

- necessary to implement safely;
- reducing a verified pain point;
- explicitly requested;
- improving maintainability with clear value.

Do not combine a feature request with an unrelated broad cleanup.

Large refactors should have explicit acceptance/verification.

---

# 76. Changes to architectural boundaries

If a task changes:

- service boundaries;
- package boundaries;
- data ownership;
- persistence model;
- externally consumed API model;
- major dependency direction;

update `ARCHITECTURE.md`.

Create an ADR when the decision is significant.

---

# 77. Changes to product scope

If the user approves a new capability:

- update `PRODUCT.md`;
- update relevant acceptance criteria;
- update `STATUS.md` when state changes.

Do not treat `IDEAS.md` as approved scope.

---

# 78. Changes to design

If UI behavior or design-system rules materially change:

- update `DESIGN.md`;
- update implementation tokens/components;
- add/adjust visual/accessibility tests where valuable.

---

# 79. Changes to quality strategy

If a new critical validation layer becomes required:

- update executable tooling;
- update CI;
- update `QUALITY.md`;
- update `AGENTS.md` only if the standard command/gate changed.

---

# 80. Changes to delivery

If deployment/release behavior changes:

- update automation;
- update `DELIVERY.md`;
- update `AGENTS.md` only when agent-visible commands or safety rules change.

---

# 81. Project bootstrap self-review

Before finalizing the bootstrap, review against these questions.

## Product

- Do we know what we are building?
- Are goals and non-goals explicit?
- Are acceptance criteria useful?

## Repository

- Is the structure conventional?
- Is setup reproducible?
- Are generated/local files handled correctly?

## Agent usability

- Can an agent discover commands immediately?
- Is `AGENTS.md` concise?
- Is information routed to canonical owners?
- Are vendor-specific files just adapters?

## Quality

- Do meaningful tests exist?
- Are checks executable?
- Is there a CI-equivalent gate?

## Safety

- Are secrets protected?
- Are destructive actions gated?
- Are user files protected?
- Is deletion policy Git-aware rather than archive-everything?

## Documentation

- Is documentation proportional?
- Is there duplication?
- Does `STATUS.md` reflect current reality?

## Delivery

- Can an artifact be built?
- Is deployment controlled?
- Is rollback described if applicable?

---

# 82. Required final bootstrap report

When finished, report to the user in a concise form.

Use a structure similar to:

```text
Bootstrap complete.

Created/updated:
- ...

Architecture:
- ...

Standard commands:
- Setup: ...
- Dev: ...
- Check: ...
- Test: ...
- Build: ...
- CI: ...

Verification:
- VERIFIED: ...
- FAILED: ...
- NOT RUN: ...
- NOT AVAILABLE: ...

Important decisions:
- ...

Continuity:
- Active contract: ...
- Last known good state: ...
- Exact next action: ...

Open questions/risks:
- ...

Next recommended step:
- ...
```

Do not dump internal reasoning.

Report decisions and evidence.

---

# 83. Research-backed rationale for this standard

This bootstrap is based on the following broad conclusions from current coding-agent and software-engineering guidance:

1. A concise repository-level agent instruction file is useful as an operational entry point.
2. `AGENTS.md` is increasingly used as a cross-agent repository instruction convention.
3. Hierarchical/local instructions are useful when a subtree genuinely has specialized rules.
4. Vendor-specific instruction files should be adapters, not divergent full copies.
5. Large persistent context files consume agent attention and should stay concise.
6. Machine-enforceable rules belong in formatters, linters, compilers, tests, and CI.
7. Architecture decision records preserve historical reasoning better than bloating current-state architecture docs.
8. Git should be used as the primary recovery mechanism for intentionally deleted tracked source.
9. Unknown untracked user files deserve stronger protection than ordinary tracked obsolete code.
10. Product intent, architecture, quality strategy, current state, and future ideas are distinct concerns and should not share one undifferentiated Markdown log.

Useful public references that informed this standard include:

- AGENTS.md open convention: https://agents.md/
- OpenAI Codex AGENTS.md guidance: https://developers.openai.com/codex/agent-configuration/agents-md
- Anthropic Claude Code project memory/instruction guidance: https://code.claude.com/docs/en/memory
- GitHub Copilot repository custom instructions: https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot
- Cursor rules guidance: https://cursor.com/docs/context/rules
- Gemini CLI project context: https://geminicli.com/docs/cli/gemini-md/
- Microsoft ADR guidance: https://learn.microsoft.com/en-us/azure/well-architected/architect-role/architecture-decision-record
- OWASP Secrets Management Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html
- Git restore/history documentation: https://git-scm.com/docs/git-restore
- Twelve-Factor App principles: https://12factor.net/

- OpenAI, "Harness engineering: leveraging Codex in an agent-first world":
  https://openai.com/index/harness-engineering/
- OpenAI, "An open-source spec for Codex orchestration: Symphony":
  https://openai.com/index/open-source-codex-orchestration-symphony/
- OpenAI, "A practical guide to building agents":
  https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/
- OpenAI, "Running Codex safely at OpenAI":
  https://openai.com/index/running-codex-safely/
- Anthropic, "Effective harnesses for long-running agents":
  https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
- Anthropic, "Scaling Managed Agents: Decoupling the brain from the hands":
  https://www.anthropic.com/engineering/managed-agents
- Gemini CLI, project context hierarchy:
  https://geminicli.com/docs/cli/gemini-md/

These references are background rationale.

Future coding agents do NOT need to browse them during ordinary work unless the relevant convention has changed or verification is necessary.

---

# 84. Bootstrap directive to the executing agent

If you are an AI coding agent reading this file because a user asked you to bootstrap the repository, execute the following now:

1. Read this entire file.
2. Inspect the workspace without destroying anything.
3. Determine whether the repository is greenfield or existing.
4. Parse the user's project description into product requirements.
5. Identify only consequential unanswered questions.
6. Ask only if a consequential ambiguity cannot safely be resolved.
7. Preserve existing stack/conventions when this is an existing repository.
8. For greenfield work, choose a mature appropriate stack.
9. Create the minimum sufficient repository structure.
10. Establish `docs/PRODUCT.md`.
11. Establish `docs/ARCHITECTURE.md`.
12. Establish `docs/QUALITY.md`.
13. Establish `docs/STATUS.md`.
14. Establish `docs/IDEAS.md`.
15. Create `docs/DESIGN.md`, `docs/SECURITY.md`, `docs/DELIVERY.md`, `docs/adr/`, and `docs/work/` only when justified.
16. Establish a small discoverable command surface for setup/dev/check/test/build/ci.
17. Add appropriate formatting/lint/type/static checks.
18. Add meaningful tests.
19. Add or integrate CI when appropriate.
20. Add safe environment configuration and `.gitignore`.
21. Build a minimal runnable/working foundation rather than only empty scaffolding when the user's request implies a new application.
22. Actually run setup/check/test/build/ci-equivalent commands that are available.
23. Fix bootstrap-created failures when possible.
24. Review all changes.
25. Generate the final concise root `AGENTS.md` from the **verified repository reality**.
26. Generate only tiny vendor adapters when needed.
27. Update `docs/STATUS.md`.
28. Report verification evidence and remaining limitations.
29. Do not declare success merely because files exist.
30. Infer and honor the user's Authorization Envelope.
31. Default to HIGH autonomy unless the user requests otherwise.
32. Do not ask for routine "continue" acknowledgements.
33. Decompose substantial work as Project → Milestone → Workstream → Contract → Atomic Unit → Evidence.
34. Maintain `docs/CONTINUITY.md` as a concise cold-resume packet.
35. Use optional `.project/WORK.json` for substantial multi-contract projects.
36. Create soft checkpoints at meaningful transitions and continue automatically.
37. Stop only for hard checkpoints, genuine external blockers, completed authorized objectives, or explicit user instruction.
38. When context pressure rises, persist state before exhaustion and use compaction/reset/continuation if the harness supports it.
39. On a cold start, verify the predecessor's claims against Git and executable checks before continuing.
40. Treat the repository—not the chat session—as durable project memory.

---

# 85. Final principle

Build repositories so that the correct behavior is obvious.

Prefer:

```text
clear product intent
+
clear architecture
+
small agent instructions
+
executable commands
+
tests
+
CI
+
safe defaults
+
single sources of truth
```

over:

```text
huge prompts
+
duplicated rules
+
manual conventions
+
agent guesswork
+
documentation drift
```

The purpose of this system is not to make an AI "remember everything."

The purpose is to make the repository itself **legible, testable, recoverable, and hard to misunderstand** for both humans and AI agents.
