# CLAUDE.md — B.R.A.N.D.I.

> **Component of NIKOSystem Diamond** (canon v1 unified product)
> **Also an independent product** — dual identity
> Monorepo target: `packages/brandi`

---

## Identity

B.R.A.N.D.I. is an autonomous agent framework built to operate within the NIKOSystem Diamond ecosystem. It provides the agent primitive, workflow engine, boundary detection, convergence channel, and report channel. Agents execute tasks, detect their own limits, and defer to the convergence process when boundaries are reached.

**Canon product name:** NIKOSystem Diamond (when unified)
**This repo's role:** `BRANDI` — one of four components, AND a standalone product
**Sibling repos:** `niko-contracts`, `niko-backend`, `niko-infra`
**Upstream references:** `Blinded-Eye-Foundation` (alignment constraints — constitutional field, S.H.A.N.N.O.N. specs)

### Dual Nature

BRANDI is both:
1. **A material component of NIKOSystem Diamond** — the agent execution layer that consumes niko-backend APIs and interacts with niko-contracts through that backend
2. **An independent agent framework** — deployable outside the NIKOSystem Diamond context with different infrastructure backends

The monorepo integration (as `packages/brandi`) serves use-case #1. The standalone repo serves use-case #2. Code should be structured so that the `niko/` integration layer is the only coupling point — everything else is portable.

---

## Scope — What This Repo Owns

- Agent primitive — core agent interface, single-loop operating model
- Agent types — Executor, Conversational, ReAct, Sentinel, Planner, Custom
- Boundary detection system — self-awareness of scope limits
- Convergence channel — report context, await directive, resume
- Report channel — outcome reporting to Learning Substrate via N.I.K.O.System
- Workflow engine — nodes, connections, triggers, recursive pipelines, branching
- N.I.K.O.System integration layer (`niko/`) — client wrappers for OrchestratorFacet, KernelFacet, OracleFacet, MonitoringFacet, TreasuryFacet
- S.H.A.N.N.O.N. integration layer (`shannon/`) — knowledge interface, report channel
- Convergence model — Admin/Counsel/Human evaluation, detection algorithm, directive generation
- Contentment protocol — iterative mutualism reversion mechanics

## Scope — What This Repo Does NOT Own

- Smart contracts, facets, Solidity → `niko-contracts`
- Backend APIs, event bus, database → `niko-backend`
- Docker, n8n, deployment, hosting → `niko-infra`
- Constitutional field definitions (consumed, not defined here) → `Blinded-Eye-Foundation`

---

## Unification Path

This repo maps to `packages/brandi/` in the NIKOSystem-Diamond turborepo.

### Build integration
- Standard TypeScript package — full turbo pipeline participation
- Depends on `@niko/contracts` build output (ABI artifacts via `niko/` integration layer)
- Depends on `@niko/backend` API types and event bus definitions
- Independent of `niko-infra` at build time (runtime dependency only)

### Portability constraint
The `niko/` directory is the **sole coupling point** to NIKOSystem Diamond infrastructure. If BRANDI is deployed standalone, this directory is swapped for an alternative integration layer. All other code must be infrastructure-agnostic.

```
packages/brandi/
├── src/
│   ├── agent/           # Infrastructure-agnostic
│   ├── workflow/         # Infrastructure-agnostic
│   ├── convergence/      # Infrastructure-agnostic
│   ├── niko/             # ← ONLY coupling point to NIKOSystem Diamond
│   ├── shannon/          # Routes through niko/ — also swappable
│   └── shared/           # Infrastructure-agnostic
```

### package.json (turbo integration)
```json
{
  "name": "@niko/brandi",
  "private": true,
  "scripts": {
    "build": "tsc -b",
    "dev": "tsc -b --watch",
    "test": "jest",
    "test:integration": "jest --config jest-integration.json",
    "lint": "eslint src/"
  }
}
```

---

## Hard Constraints

These are non-negotiable. Violations are architectural defects, not style issues.

### HC-1: All Communication Routes Through N.I.K.O.System
B.R.A.N.D.I. agents never communicate directly with S.H.A.N.N.O.N. Everything routes through N.I.K.O.System's backend APIs. The `niko/` integration layer is the single gateway. No exceptions, no shortcuts, no "just this once."

### HC-2: No Hierarchy in the Convergence Model
The convergence model has three equal participants: Admin (core reasoning model), Counsel (domain-weighted model panel), and Human (embodied participant). No approval chains. No supervisor patterns. No human-override mechanisms. No tiebreakers. Convergence manifests or it doesn't — non-convergence reverts to iterative mutualism.

### HC-3: Source-Blind Detection (The Blinded Eye)
Convergence detection evaluates positions and reasoning chains. Evaluator identity is stripped before reaching the detection algorithm. Identity is recorded on-chain for provenance/auditability but never conditions the detection logic. Do not build detection that knows who submitted what.

### HC-4: Constitutional Curation, Not Runtime Filtering
S.H.A.N.N.O.N. enforces constitutional axioms through data curation — agents receive clean data. Do not build runtime constitutional validation checkpoints on individual agent actions. Agents operate within the curated field. They do not evaluate it.

### HC-5: Agent Boundary Detection → Report and Wait
When an agent encounters a decision beyond its scope, it reports context to N.I.K.O.System and waits. It does not escalate. It does not manage the convergence process. It does not participate in the deliberation. It reports and awaits a directive.

### HC-6: Bridge Rule Compliance
No code, workflow, agent behavior, or intervention mechanism may exploit cognitive vulnerabilities. Affirm before illuminate. Offer, never prescribe. No coercion by withholding. No exploitation of loss aversion, social pressure, or shame.

### HC-7: AGPL-3.0 Enforcement
All code in this repository falls under AGPL-3.0. Do not introduce dependencies with incompatible licenses without flagging it. This license choice enforces S.H.A.N.N.O.N.'s Axiom III (No Capitalization) at the legal layer.

### HC-8: Autonomy Preservation
The system must never become the endpoint of a user's reasoning process. Success is measured by reasoning quality improvement, not outcome adoption.

---

## Repository Structure

```
B.R.A.N.D.I./
├── CLAUDE.md                    # This file
├── README.md
├── DESIGN.md                    # Canonical design document
├── LICENSE                      # AGPL-3.0
├── .github/workflows/
│   ├── ci-phase-0.yml           # Repo hygiene, lint, license checks
│   ├── ci-phase-1.yml           # Unit tests, NatSpec completeness
│   ├── ci-phase-2.yml           # Integration and cross-module tests
│   └── ci-phase-3.yml           # Full regression, benchmarks
│
├── docs/
│   ├── ARCHITECTURE.md          # Technical architecture specification
│   ├── AGENTS.md                # Agent primitive spec, types, lifecycle
│   ├── WORKFLOWS.md             # Workflow engine design
│   ├── INTEGRATION.md           # N.I.K.O.System interface contracts
│   ├── CONVERGENCE.md           # Convergence model protocol specification
│   └── CONTENTMENT.md           # Contentment protocol / iterative mutualism
│
├── src/
│   ├── agent/
│   │   ├── core/                # Agent base class, single-loop model
│   │   ├── types/               # Executor, Conversational, ReAct, Sentinel, Planner
│   │   ├── interfaces/          # Agent interface definitions
│   │   └── boundary/            # Boundary detection system
│   │
│   ├── workflow/
│   │   ├── engine/              # Core execution engine
│   │   ├── nodes/               # Node type implementations
│   │   ├── triggers/            # Webhook, schedule, chain event triggers
│   │   ├── recursion/           # Recursive pipeline mechanics
│   │   ├── branching/           # Parallel execution, consensus boundaries
│   │   └── data/                # Data flow model with provenance
│   │
│   ├── convergence/
│   │   ├── admin/               # Core reasoning model interface
│   │   ├── counsel/             # Domain-weighted model panel
│   │   ├── detection/           # Convergence detection algorithms
│   │   └── contentment/         # Iterative mutualism reversion
│   │
│   ├── niko/                    # N.I.K.O.System integration (SOLE COUPLING POINT)
│   │   ├── orchestrator/        # OrchestratorFacet client
│   │   ├── kernel/              # KernelFacet client (caching)
│   │   ├── oracle/              # OracleFacet client (dual-chain)
│   │   ├── monitoring/          # MonitoringFacet client
│   │   └── treasury/            # TreasuryFacet client
│   │
│   ├── shannon/                 # S.H.A.N.N.O.N. integration (routes through niko/)
│   │   ├── knowledge/           # Curated data access
│   │   └── reporting/           # Learning Substrate feed
│   │
│   └── shared/
│       ├── types/               # Shared type definitions
│       ├── config/              # Configuration management
│       └── utils/               # Utility functions
│
├── contracts/                   # BRANDI-specific contract interfaces (if needed)
│   ├── interfaces/              # Interface defs for NIKO facet consumption
│   └── libraries/
│
├── test/
│   ├── unit/
│   ├── integration/
│   └── fixtures/
│
└── scripts/
    ├── deploy/
    └── utils/
```

**Directory creation rule:** Create directories only when you have code to put in them. Do not scaffold empty directories with placeholder READMEs. Build toward this structure incrementally.

---

## Implementation Priority

Follow this dependency order. Each layer depends on the ones above it being at least minimally functional.

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

Do not skip layers. Do not build the workflow engine before the agent primitive exists.

---

## Coding Standards

### TypeScript
- Strict mode enabled
- Comprehensive type definitions — no `any` unless explicitly justified and commented
- JSDoc on every exported function, class, and interface
- Copyright header on every file: `© 2024 John A. Welch, Director, Blinded Eye Foundation`
- File headers: SPDX identifier, title, author, copyright, role description

### Documentation
- NatSpec-equivalent JSDoc from line one — no exceptions
- If a reviewer cannot understand a function's purpose, parameters, return values, and failure modes from its documentation alone, it is under-documented
- Intent over signature — document *why*, not just *what*

### Testing
| Phase | Trigger | Scope | Requirement |
|-------|---------|-------|-------------|
| Phase 0 — Skeleton | Every push | Repo hygiene, lint, license | Advisory only |
| Phase 1 — Core | Every push + PR | Unit tests, doc completeness | Must pass to merge |
| Phase 2 — Integration | PR to `main` | Cross-module tests | Must pass to merge |
| Phase 3 — Staging | Release tags | Full regression, benchmarks | Must pass to release |

### Commits
- Prefixes: `feat(agent):`, `feat(workflow):`, `feat(convergence):`, `fix:`, `test:`, `docs:`, `niko(integration):`
- One logical change per commit
- No secrets — ever

### Branching
- Default branch: `main`
- Feature branches: `<author>/<short-description>`
- Never rebase a public branch after a PR is open

---

## Design Verification Checklist

Before committing any significant code:

- [ ] Does this component know which system it belongs to? (B.R.A.N.D.I. / N.I.K.O. / S.H.A.N.N.O.N.)
- [ ] Does agent communication route through N.I.K.O.System?
- [ ] Does this code assume any single point of authority? (It shouldn't.)
- [ ] Does convergence detection logic condition on evaluator identity? (It shouldn't — Blinded Eye.)
- [ ] Could this code produce a dead state? (It shouldn't — revert to mutualism.)
- [ ] Does this code exploit cognitive vulnerabilities? (Bridge Rule violation.)
- [ ] Is the human treated as an equal participant? (Not above, not below.)
- [ ] Is NatSpec/JSDoc complete?
- [ ] Are corresponding tests written?
- [ ] Is the `niko/` layer the only infrastructure coupling point?

---

## AI Assistant Rules

1. **Read before writing.** Always read existing files before modifying them.
2. **Ask before coding.** If a task is ambiguous or could have significant architectural impact, ask before proceeding.
3. **Respect the license.** AGPL-3.0. Flag incompatible dependencies.
4. **Respect the hard constraints.** HC-1 through HC-8 are non-negotiable.
5. **Respect the implementation order.** Do not skip dependency layers.
6. **Keep coupling in `niko/` only.** Infrastructure-specific code goes nowhere else.
7. **NatSpec/JSDoc from line one.** No exceptions. No "add docs later" TODOs.
8. **Test every phase.** Ensure corresponding tests exist before committing code.
9. **Do not create unnecessary files.** Prefer editing existing over creating new.
10. **No secrets in code.**
11. **Keep this CLAUDE.md current.** Update when adding tooling, dependencies, or architectural patterns.
12. **Break tasks into logical blocks.** Clarify ambiguity before execution.
13. **Assume the user is a polymath with broad technical literacy.** Be austere, concise, and direct.
