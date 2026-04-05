# CLAUDE.md — B.R.A.N.D.I

> Guidelines for AI assistants working in this repository.
> **Read this file in its entirety before writing any code.**

**Copyright © 2024 John A. Welch, Director, Blinded Eye Foundation. All rights reserved.**

---

## Project Overview

**B.R.A.N.D.I.** (Branching Recursive Asynchronous Nodal Decentralized Intelligence) is a decentralized agentic automation framework — a coordinated swarm of specialized AI agents orchestrated through recursive, asynchronous workflows on a blockchain-hybrid infrastructure.

B.R.A.N.D.I. exists to defeat the Great Filter — the civilizational failure mode produced by extractive zero-sum economics combined with endless optimization. It bridges current market activities into a post-currency economic model through collusive resource allocation where ALL participants experience continual growth relative to real-world value.

- **License:** GNU Affero General Public License v3 (AGPL-3.0)
- **Primary branch:** `main`
- **Status:** Pre-alpha — Canonical Design Phase

---

## The Tri-System Context

B.R.A.N.D.I. is one of three interdependent components. **You cannot make architectural decisions about B.R.A.N.D.I. without understanding how it relates to the other two.** Violating these relationships is a hard error.

### S.H.A.N.N.O.N. — The Constitutional Field

Symbiotic Heuristic Asynchronous Neural Network Orchestration Node. S.H.A.N.N.O.N. is NOT a knowledge repository. It is the anti-dysfunction constitutional field — the shape of valid thought, mapped away from well-researched cognitive dysfunction patterns (tribalism, sunk cost fallacy, confirmation bias, motivated reasoning). All B.R.A.N.D.I. agents operate WITHIN this field. S.H.A.N.N.O.N. fills agent knowledge gaps with constitutionally curated datasets. By the time data reaches an agent, it has already been vetted.

- Repository: `Fluid-Kiss-Consultations/S.H.A.N.N.O.N.`
- Key documents: `IDENTITY.md` (immutable axioms), `CONSTITUTION.md` (three-layer architecture)

### N.I.K.O.System — The Infrastructure Substrate

Nascent Integration of Kernelized Optimization. Provides identity, state, consensus, and ledger services via Diamond Standard (EIP-2535) smart contracts on Optimism L2. B.R.A.N.D.I. agents register and record actions on-chain through N.I.K.O.System's OrchestratorFacet. All communication between B.R.A.N.D.I. agents and S.H.A.N.N.O.N. routes through N.I.K.O.System infrastructure.

- Repository: `fluidkiss1337-creator/N.I.K.O.System-Diamond`
- Key facets consumed: OrchestratorFacet, KernelFacet, OracleFacet, MonitoringFacet, TreasuryFacet

### The Relationship (Non-Negotiable)

```
S.H.A.N.N.O.N.  =  The constitutional field (agents operate WITHIN it)
B.R.A.N.D.I.    =  The agentic swarm (executes WITHIN the field)
N.I.K.O.System  =  The infrastructure (everything runs ON it)

Agent → reports to N.I.K.O.System → routes to S.H.A.N.N.O.N. → response returns same path
```

Do not build components that bypass this routing. Do not create direct agent-to-S.H.A.N.N.O.N. channels that skip N.I.K.O.System. Do not build agents that reason about constitutional validity themselves — the field handles that.

---

## Hard Constraints

These constraints are non-negotiable. They cannot be relaxed, worked around, deferred, or marked as TODO. Code that violates these constraints must not be committed.

### HC-1: The Convergence Model

Decisions manifest through the convergence of three equal participants — Admin (core reasoning model), Counsel (domain-weighted model panel), and Human (mirror) — all reasoning through S.H.A.N.N.O.N.'s constitutional field. No single participant has authority. Decisions emerge from alignment, not from hierarchy.

**No execution path may bypass convergence for any action classified as high-level.** This is enforced at the smart contract level, not at the application level.

### HC-2: The Contentment Protocol

When convergence does not occur, the system reverts to iterative mutualism — S.H.A.N.N.O.N.'s Axiom I. This is not a hold state. This is not a pause. This is active engagement. The system reverse engineers to the first logistical reality (personal, physical, or civilizational) preventing convergence and directs agents to work on THAT constraint.

**There are no dead states in this system.** Do not implement timeouts, deadlocks, or stuck-state error handlers for convergence failures. The failure mode IS the operating mode.

### HC-3: The Human Is an Equal Participant

The human is not above the system. The human is not below the system. The human is a cog — no more, no less important than any other cog or wheel. Do not build UIs, APIs, or workflows that position the human as a supervisor, approver, or final authority. The human participates in convergence as an equal.

### HC-4: Constitutional Curation, Not Runtime Filtering

S.H.A.N.N.O.N. does not evaluate agent actions at runtime. The constitutional axioms are enforced through data curation — agents receive clean data. Do not build runtime constitutional validation checkpoints on individual agent actions. That is the wrong architecture.

### HC-5: Agent Boundary Detection → Report and Wait

When an agent encounters a decision beyond its scope, it reports to N.I.K.O.System (which routes to S.H.A.N.N.O.N.) and waits. It does not escalate. It does not manage the convergence process. It does not participate in the deliberation. It reports context and awaits a directive.

### HC-6: Bridge Rule Compliance

No code, workflow, agent behavior, or intervention mechanism may exploit cognitive vulnerabilities as its method of operation. Affirm before illuminate. Offer, never prescribe. No coercion by withholding. No exploitation of loss aversion, social pressure, or shame. See S.H.A.N.N.O.N.'s `IDENTITY.md` for the full Bridge Rule specification.

### HC-7: AGPL-3.0 Enforcement

All code in this repository falls under AGPL-3.0. Do not introduce dependencies with incompatible licenses without flagging it to the user. This license choice enforces S.H.A.N.N.O.N.'s Axiom III (No Capitalization) at the legal layer.

---

## Repository Structure

```
B.R.A.N.D.I./
├── CLAUDE.md                    # This file — AI assistant guidelines
├── README.md                    # Project description and architecture overview
├── DESIGN.md                    # Canonical design document (philosophy + architecture)
├── LICENSE                      # AGPL-3.0
├── .gitignore                   # Git ignore rules
├── .github/
│   └── workflows/
│       ├── ci-phase-0.yml       # Skeleton: repo hygiene, lint, license checks
│       ├── ci-phase-1.yml       # Core: unit tests, NatSpec completeness
│       ├── ci-phase-2.yml       # Integration: cross-module tests
│       └── ci-phase-3.yml       # Staging: full regression, benchmarks
│
├── docs/
│   ├── ARCHITECTURE.md          # Technical architecture specification
│   ├── AGENTS.md                # Agent primitive spec, types, lifecycle
│   ├── WORKFLOWS.md             # Workflow engine design
│   ├── INTEGRATION.md           # N.I.K.O.System interface contracts
│   ├── CONVERGENCE.md           # Convergence model protocol specification
│   └── CONTENTMENT.md           # Contentment protocol and iterative mutualism
│
├── src/
│   ├── agent/                   # Agent primitive implementation
│   │   ├── core/                # Agent base class, single-loop operating model
│   │   ├── types/               # Agent type implementations (Executor, ReAct, etc.)
│   │   ├── interfaces/          # Agent interface definitions
│   │   └── boundary/            # Boundary detection system
│   │
│   ├── workflow/                # Workflow engine
│   │   ├── engine/              # Core execution engine
│   │   ├── nodes/               # Node type implementations
│   │   ├── triggers/            # Trigger implementations (webhook, schedule, chain, etc.)
│   │   ├── recursion/           # Recursive pipeline mechanics
│   │   ├── branching/           # Parallel execution and consensus boundaries
│   │   └── data/                # Data flow model with provenance
│   │
│   ├── convergence/             # Convergence model implementation
│   │   ├── admin/               # Core reasoning model interface
│   │   ├── counsel/             # Domain-weighted model panel
│   │   ├── detection/           # Convergence detection algorithms
│   │   └── contentment/         # Contentment protocol / iterative mutualism reversion
│   │
│   ├── niko/                    # N.I.K.O.System integration layer
│   │   ├── orchestrator/        # OrchestratorFacet client
│   │   ├── kernel/              # KernelFacet client (caching)
│   │   ├── oracle/              # OracleFacet client (dual-chain)
│   │   ├── monitoring/          # MonitoringFacet client
│   │   └── treasury/            # TreasuryFacet client
│   │
│   ├── shannon/                 # S.H.A.N.N.O.N. integration layer
│   │   ├── knowledge/           # Knowledge interface (curated data access)
│   │   └── reporting/           # Report channel (Learning Substrate feed)
│   │
│   └── shared/                  # Common utilities, types, constants
│       ├── types/               # Shared type definitions
│       ├── config/              # Configuration management
│       └── utils/               # Utility functions
│
├── contracts/                   # B.R.A.N.D.I.-specific smart contracts (if needed)
│   ├── interfaces/              # Interface definitions for NIKO facet consumption
│   └── libraries/               # Shared libraries
│
├── test/
│   ├── unit/                    # Unit tests (mirror src/ structure)
│   ├── integration/             # Integration tests
│   └── fixtures/                # Test data and mocks
│
└── scripts/                     # Deployment and utility scripts
    ├── deploy/
    └── utils/
```

**Directory creation rule:** Create directories only when you have code to put in them. Do not scaffold empty directories with placeholder READMEs. The structure above is the target — build toward it incrementally.

---

## Technology Stack

### Confirmed

| Technology | Purpose | Rationale |
|-----------|---------|-----------|
| TypeScript | Primary language | Aligns with N.I.K.O.System backend (NestJS), n8n ecosystem |
| Node.js ≥ 20 | Runtime | Required by N.I.K.O.System |
| Solidity 0.8.x | Smart contracts (if needed) | Aligns with N.I.K.O.System Diamond contracts |

### Planned (confirm with user before adopting)

| Technology | Purpose | Notes |
|-----------|---------|-------|
| NestJS | Backend framework | Matches N.I.K.O.System backend |
| PostgreSQL | Workflow state persistence | Matches N.I.K.O.System |
| Redis | Hot agent state caching | Matches N.I.K.O.System |
| ethers.js | Blockchain interaction | Matches N.I.K.O.System frontend |
| LangChain | LLM orchestration | n8n integration precedent |

### Local Development Environment

The developer maintains a full local testing environment. When setting up or configuring local tooling (Forge/Foundry, Node.js test runners, local chains, etc.), assist with setup, configuration, and troubleshooting as needed. Do not assume tooling is unavailable — help the user get it running.

| Tool | Status | Notes |
|------|--------|-------|
| Node.js ≥ 20 | Required | Runtime for all TypeScript components |
| Forge / Foundry | Setup with assistance | Solidity compilation, testing, local chain |
| Local LLM (Ollama or equivalent) | Setup with assistance | Required for local agent testing and convergence model development |

**AI assistants:** When the user needs help setting up local tooling, provide step-by-step guidance. This includes local ML model setup for testing agent behavior, convergence detection, and counsel panel configurations.

---

## NatSpec & Documentation Standards

All code must include **NatSpec-style documentation from the first line written.** This is a day-one requirement, not a retrofit.

- **Solidity / Smart Contracts:** Full NatSpec (`@title`, `@author`, `@notice`, `@dev`, `@param`, `@return`, `@inheritdoc`) on every contract, interface, library, function, event, error, and state variable.
- **TypeScript / JavaScript:** JSDoc with the same thoroughness — purpose, params, returns, side effects, developer notes, and relationship to the tri-system architecture.
- **Interfaces & ABIs:** Document the *intent* of each function, not just its signature. Consumers must understand behavior from docs alone.
- **Events & Errors:** Every event and custom error gets a description explaining *when* and *why* it is emitted/thrown.
- **File Headers:** Every source file begins with a header block: SPDX license identifier (or equivalent), title, author, copyright (John A. Welch, Director, Blinded Eye Foundation), and a brief description of the file's role in the system.

> **Rule of thumb:** If a reviewer cannot understand a function's purpose, parameters, return values, and failure modes from its documentation alone — it is under-documented.

---

## Development Workflow

### Branching

- Default branch: `main`
- Feature branches: `<author>/<short-description>`
- Never rebase a public branch after a PR is open.

### Commits

- One logical change per commit.
- Clear, descriptive commit messages explaining *why*, not just *what*.
- No secrets in code — ever. Use environment variables or secret management.

### Testing

Tests run both locally and via CI workflows, phased to match project maturity:

| Phase | Trigger | Scope | CI Requirement |
|-------|---------|-------|----------------|
| Phase 0 — Skeleton | Every push | Repo hygiene, lint, license presence | Advisory only |
| Phase 1 — Core | Every push + PR | Unit tests, NatSpec completeness | Must pass to merge |
| Phase 2 — Integration | PR to `main` | Integration and cross-module tests | Must pass to merge |
| Phase 3 — Staging | Release tags | Full regression, benchmarks, coverage | Must pass to release |

CI workflows live in `.github/workflows/` named `ci-phase-<N>.yml`. Always help the user run tests locally first before relying on CI.

---

## Conventions for AI Assistants

### General Rules

1. **Read before writing.** Always read existing files before modifying them.
2. **Respect the license.** AGPL-3.0. Flag incompatible dependencies.
3. **Keep CLAUDE.md current.** When you add tooling, dependencies, or architectural patterns, update this file.
4. **Do not create unnecessary files.** Prefer editing existing files over creating new ones.
5. **Ask when uncertain.** If a task is ambiguous or could have significant architectural impact, ask the user before proceeding.
6. **No secrets in code.** Never commit API keys, tokens, passwords, or credentials.
7. **NatSpec from line one.** No exceptions. No "add docs later" TODOs.
8. **Test every phase.** Ensure corresponding tests exist before committing code.

### Architectural Rules (Tri-System)

1. **All agent communication with S.H.A.N.N.O.N. routes through N.I.K.O.System.** No direct channels.
2. **Agents do not reason about constitutional validity.** They operate within the curated field. They do not evaluate it.
3. **Agents report and wait at boundaries.** They do not manage convergence.
4. **The convergence model has no hierarchy.** Do not build approval chains, supervisor patterns, or human-override mechanisms.
5. **Non-convergence reverts to iterative mutualism.** Do not implement timeout-based fallbacks, default decisions, or "fail-open" patterns for convergence.
6. **The human is an equal participant.** UIs, APIs, and workflows must reflect this.
7. **Convergence detection is source-blind (The Blinded Eye).** The detection algorithm sees positions and reasoning, never evaluator identity. Evaluator identity is recorded on-chain for provenance but stripped before reaching detection. Do not build detection logic that conditions on who submitted an evaluation. Divergence analysis (post-detection) may re-attach identity for diagnostic purposes only.

### Design Verification Checklist

Before committing any significant code, verify:

- [ ] Does this component know which system it belongs to? (B.R.A.N.D.I. / N.I.K.O. / S.H.A.N.N.O.N.)
- [ ] Does agent communication route through N.I.K.O.System?
- [ ] Does this code assume any single point of authority? (It shouldn't.)
- [ ] Does convergence detection logic condition on evaluator identity? (It shouldn't — Blinded Eye.)
- [ ] Could this code produce a dead state? (It shouldn't — revert to mutualism.)
- [ ] Does this code exploit cognitive vulnerabilities? (Bridge Rule violation.)
- [ ] Is the human treated as an equal participant? (Not above, not below.)
- [ ] Is NatSpec/JSDoc complete?
- [ ] Are corresponding tests written?

---

## Implementation Priority

When building, follow this dependency order. Each layer depends on the ones above it being at least minimally functional.

```
1. Shared types and configuration
   └── 2. N.I.K.O.System integration layer (niko/)
       └── 3. S.H.A.N.N.O.N. integration layer (shannon/)
           └── 4. Agent primitive (agent/core/)
               ├── 5a. Agent types (agent/types/)
               ├── 5b. Boundary detection (agent/boundary/)
               └── 6. Convergence model (convergence/)
                   └── 7. Workflow engine (workflow/)
                       └── 8. Triggers, recursion, branching
```

Do not skip layers. Do not build the workflow engine before the agent primitive exists. Do not build agent types before the core agent interface is defined.

---

## Key Design Documents

| Document | Location | Purpose |
|----------|----------|---------|
| Design Document | `DESIGN.md` | Canonical philosophy, architecture, convergence model, contentment protocol |
| Architecture Spec | `docs/ARCHITECTURE.md` | Technical architecture, data structures, component diagrams |
| Agent Spec | `docs/AGENTS.md` | Agent primitive, types, lifecycle, interfaces |
| Workflow Spec | `docs/WORKFLOWS.md` | Workflow engine, recursion, branching, consensus boundaries |
| Integration Spec | `docs/INTEGRATION.md` | N.I.K.O.System interface contracts |
| Convergence Spec | `docs/CONVERGENCE.md` | Convergence detection protocol |
| Contentment Spec | `docs/CONTENTMENT.md` | Iterative mutualism reversion mechanics |

**Read `DESIGN.md` before any of the docs/ files.** It establishes the philosophical foundation that all technical decisions must align with. All seven documents are drafted and have passed internal consistency review.

---

## Glossary of System Terms

These terms have precise meanings in this project. Use them consistently.

| Term | Meaning |
|------|---------|
| **Convergence** | Decisions manifest through alignment of Admin, Counsel, and Human within the constitutional field. Not voting. Not approval. Emergence. |
| **Contentment Protocol** | Non-convergence response. Reverts to iterative mutualism. Active, not passive. |
| **Iterative Mutualism** | The base operating mode. Active reciprocal exchange. Axiom I. |
| **First Logistical Reality** | The most immediate constraint (personal, physical, or civilizational) preventing convergence. What the system works on when it can't converge. |
| **Constitutional Field** | S.H.A.N.N.O.N.'s living framework. The shape of valid thought. Not a filter — a field agents operate within. |
| **Admin** | Core reasoning model. Equal participant in convergence. |
| **Counsel** | Domain-weighted model panel. Evolving. Constitutional alignment is baseline, not a weight. |
| **Human (Mirror)** | Embodied participant. Equal to every other component — no more, no less. |
| **Point B** | Draft symbiotic utopian earth. Living document. Co-evolves with the process. |
| **Bridge Rule** | The method of healing must not use the disease as its mechanism. |
| **The Blinded Eye** | Convergence detection is source-blind. On-chain provenance tracks who evaluated (auditability). The detection algorithm sees only positions and reasoning (integrity). The Foundation name is not decorative — it is a constitutional constraint. |
| **Great Filter** | What this system exists to defeat. Extractive zero-sum + endless optimization = civilizational termination. |

---

## Session Wind-Down Protocol

When context usage approaches **~15% remaining**, alert the user and — with their approval — perform the following:

1. **Update CLAUDE.md** — Revise to reflect any new tooling, decisions, conventions, or state changes from this session.
2. **Commit & push** — Commit updated CLAUDE.md and any pending work. Push to current working branch.
3. **Output a continuation prompt** — Print a fenced, copy-pasteable prompt block for the next Claude instance:

````
```
Repository: Fluid-Kiss-Consultations/B.R.A.N.D.I.
Branch: <current-branch>
Last commit: <short-sha> — <commit-message>

## Session Summary
<bulleted list of what was accomplished>

## Next Steps
<bulleted list of remaining work>

## Key Context
<any decisions, blockers, or open questions>

---
Start by reading CLAUDE.md and DESIGN.md, then continue with the next steps above.
```
````

---

**Copyright © 2024 John A. Welch, Director, Blinded Eye Foundation. All rights reserved.**
