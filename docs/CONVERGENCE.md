# B.R.A.N.D.I. — Convergence Detection Protocol Specification

**Copyright © 2024 John A. Welch, Director, Blinded Eye Foundation. All rights reserved.**

**Prerequisites:** Read `DESIGN.md`, `docs/ARCHITECTURE.md`, and `docs/INTEGRATION.md` before this document.

---

## 1. What Convergence Is (And Is Not)

Convergence is the condition in which Admin, Counsel, and Human — all reasoning through S.H.A.N.N.O.N.'s constitutional field — arrive at the same decision point within the valid decision space. The decision is not chosen by any party. It is not voted on. It is not averaged. It **manifests** — the point where all perspectives, weighted differently and arriving from different domains, independently land on the same answer.

**Convergence is not:**

- **Voting.** There is no majority rule. Two parties cannot outvote the third. All three must align.
- **Approval.** No party approves or rejects the others' positions. Each evaluates independently within the constitutional field.
- **Compromise.** The convergence point is not a midpoint between positions. It is the point that satisfies all constitutional constraints across all evaluating parties simultaneously. If that point does not exist, convergence does not occur — and that is an acceptable outcome (see CONTENTMENT.md).
- **Consensus-seeking.** The system does not push parties toward agreement. It presents the constitutional space, lets each party reason within it, and checks whether their conclusions overlap. If they do not, the system does not try harder. It reverts to iterative mutualism.

**The Blinded Eye.** Convergence detection is source-blind. Each evaluator is individually identified on-chain for auditability and provenance — the system knows who evaluated, and that record is immutable. But the detection algorithm itself never sees who submitted an evaluation. It sees positions, reasoning chains, and parameters. It judges the reasoning, never the reasoner. The Foundation name is not decorative. This is a constitutional constraint: the method of judgment must not use the identity of the judge as its mechanism.

---

## 2. The Evaluation Process

When a convergence request arrives (submitted by an agent through N.I.K.O.System, per INTEGRATION.md), the Convergence Module initiates a structured evaluation cycle.

### 2.1 Evaluation Lifecycle

```
ConvergenceRequest received
         │
         ▼
    ┌─────────────────────┐
    │  DISTRIBUTE          │
    │                      │
    │  Send DecisionContext│
    │  to all evaluators   │
    │  simultaneously      │
    └──────────┬───────────┘
               │
    ┌──────────▼───────────┐
    │  EVALUATE             │
    │                      │
    │  Admin evaluates     │
    │  Counsel evaluates   │
    │  (each domain        │
    │   independently)     │
    │  Human evaluates     │
    │                      │
    │  All reasoning       │
    │  through the same    │
    │  constitutional field│
    └──────────┬───────────┘
               │
    ┌──────────▼───────────┐
    │  DETECT               │
    │                      │
    │  Compare evaluations │
    │  Run convergence     │
    │  detection algorithm │
    └────┬─────────────┬───┘
         │             │
    CONVERGED    NOT CONVERGED
         │             │
         ▼             ▼
    Directive     Revert to
    manifests     Iterative
                  Mutualism
                  (CONTENTMENT.md)
```

### 2.2 Evaluation Simultaneity

All evaluators receive the `DecisionContext` at the same time. No evaluator sees another's evaluation before submitting its own. This prevents anchoring bias — a cognitive dysfunction where early inputs disproportionately influence later reasoning. The system enforces temporal isolation between evaluators during the evaluation phase.

The integration layer enforces this by:

1. Publishing the `DecisionContext` to all evaluator channels simultaneously via the Event Bus.
2. Accepting evaluations into a sealed store that does not reveal any evaluation until all have been received (or the evaluation window closes).
3. Only after all evaluations are received (or the window closes) does the detection algorithm run.

### 2.3 Evaluation Window

Evaluations are not open-ended. Each convergence request has an evaluation window — a period during which evaluations must be submitted. The window duration varies by impact level.

| Impact Level | Evaluation Window | Rationale |
|-------------|-------------------|-----------|
| LOCAL | 5 minutes | Routine scope decision — fast turnaround expected |
| CLUSTER | 15 minutes | Affects multiple agents — more consideration needed |
| SWARM | 1 hour | Broad system impact — Counsel domains need time to evaluate |
| EXTERNAL | 4 hours | Affects outside participants — Human needs time for physical-world consultation |
| CIVILIZATIONAL | 24 hours | Trajectory-altering — all parties need deep evaluation |

**Window expiration does not produce a timeout error.** If not all evaluations arrive within the window, the system does not fail. It reverts to iterative mutualism with the missing evaluation identified as a logistical reality to address (see CONTENTMENT.md §3). The missing evaluator's capacity to participate becomes the first logistical reality.

---

## 3. Evaluation Submission Format

Each evaluator submits a structured `Evaluation` object. The format is identical for Admin, Counsel (each domain), and Human. The system does not distinguish evaluations by source during detection — it examines the reasoning topology, not the identity of the evaluator.

### 3.1 Evaluation Structure

```typescript
/**
 * @title Evaluation
 * @notice A single evaluator's assessment of a decision within
 *         the constitutional field.
 * @dev All evaluations use the same structure regardless of
 *      evaluator type. The convergence detection algorithm
 *      operates on the structure, not on evaluator identity.
 */
interface Evaluation {
  /** The evaluating party identifier */
  evaluator: EvaluatorId;

  /** The decision point this evaluation addresses */
  decisionHash: string;

  /**
   * The evaluator's position — a structured description of
   * the decision they arrived at within the constitutional space.
   * This is not a vote. It is a description of the point in the
   * decision space where the evaluator's reasoning landed.
   */
  position: DecisionPosition;

  /**
   * Structured reasoning chain — how the evaluator arrived
   * at this position. Required for the Learning Substrate
   * and for divergence analysis.
   */
  reasoning: ReasoningChain;

  /**
   * Confidence level (0.0 to 1.0).
   * This is NOT used to weight the evaluation.
   * It is metadata for the Learning Substrate — how certain
   * the evaluator is in its own reasoning.
   * Low confidence + correct position is still valid.
   * High confidence + incorrect position is still wrong.
   */
  confidence: number;

  /** Domains the evaluator considered in reaching this position */
  domainsConsidered: string[];

  /** Constitutional hash at time of evaluation */
  constitutionalHash: string;

  /** Timestamp of submission */
  evaluatedAt: number;
}

type EvaluatorId =
  | { type: 'admin' }
  | { type: 'counsel'; domain: string }
  | { type: 'human' };
```

### 3.2 Decision Position

The `DecisionPosition` is the core of the evaluation — the point in the decision space where the evaluator landed.

```typescript
/**
 * @title DecisionPosition
 * @notice Describes a specific point in the constitutional decision space.
 * @dev Positions are compared structurally during convergence detection.
 *      Two positions converge if they describe the same action with
 *      compatible parameters within constitutional bounds.
 */
interface DecisionPosition {
  /**
   * The action the evaluator concludes should be taken.
   * Expressed as a structured action type, not free text.
   */
  action: DirectiveActionType;

  /**
   * Parameters for the action. The structure depends on the
   * action type. Parameters are compared during convergence
   * detection — they must be compatible, not identical.
   */
  parameters: Record<string, unknown>;

  /**
   * Constitutional constraints the evaluator identified as
   * relevant to this decision. Used for divergence analysis —
   * if evaluators disagree, knowing which constraints they
   * prioritized helps identify the divergence point.
   */
  relevantConstraints: string[];

  /**
   * Domains affected by this position.
   * Used for Counsel cross-checking — if a position affects
   * a domain, the Counsel member for that domain should have
   * considered it.
   */
  affectedDomains: string[];

  /**
   * Risk assessment from this evaluator's perspective.
   * Not used for convergence detection — used for the
   * Learning Substrate and divergence analysis.
   */
  riskAssessment: RiskLevel;
}

type DirectiveActionType = 'execute' | 'delegate' | 'spawn' | 'abort' | 'modify_scope';

enum RiskLevel {
  MINIMAL = 'minimal',
  LOW = 'low',
  MODERATE = 'moderate',
  HIGH = 'high',
  CRITICAL = 'critical',
}
```

### 3.3 Reasoning Chain

Every evaluation must include its reasoning chain. This is not optional. Positions without reasoning are rejected.

```typescript
/**
 * @title ReasoningChain
 * @notice The structured reasoning path from decision context to position.
 * @dev Required for divergence analysis and Learning Substrate ingestion.
 *      The system cannot learn from convergence or non-convergence
 *      without understanding HOW each evaluator reasoned.
 */
interface ReasoningChain {
  /** Ordered sequence of reasoning steps */
  steps: ReasoningStep[];

  /**
   * Constitutional axioms invoked during reasoning.
   * Maps axiom identifier to how it influenced the reasoning.
   */
  axiomsInvoked: Record<string, string>;

  /**
   * Knowledge gaps identified during reasoning.
   * These feed the Learning Substrate even if convergence occurs —
   * the system should know what knowledge was missing.
   */
  knowledgeGaps: string[];
}

interface ReasoningStep {
  /** What was considered at this step */
  consideration: string;

  /** What conclusion was drawn */
  conclusion: string;

  /** What evidence or knowledge supported this step */
  evidence: string;

  /** Was knowledge from S.H.A.N.N.O.N. used at this step? */
  usedCuratedKnowledge: boolean;
}
```

---

## 4. Convergence Detection Algorithm

The detection algorithm determines whether the submitted evaluations converge — whether all evaluators have landed on the same point in the constitutional decision space.

### 4.1 Algorithm Overview

```
Input: Set of Evaluations from all evaluators
Output: ConvergenceCheckResult (converged + directive OR not converged + divergence points)

Phase 1: COMPLETENESS CHECK
  - Are all required evaluations present?
  - Admin: exactly 1 evaluation
  - Counsel: at least 1 evaluation per active domain
  - Human: exactly 1 evaluation
  - If incomplete → revert to mutualism (missing evaluator is the logistical reality)

Phase 2: CONSTITUTIONAL CONSISTENCY
  - Do all evaluations reference the same constitutionalHash?
  - If the constitutional field evolved during evaluation, the evaluation
    cycle must restart. The evaluators were reasoning in different spaces.

Phase 3: ACTION ALIGNMENT
  - Do all evaluations propose the same action type?
  - If Admin says 'execute' and Human says 'abort', they diverge on action.
  - Divergence on action type is the strongest form of non-convergence.

Phase 4: PARAMETER COMPATIBILITY
  - If action types align, are the parameters compatible?
  - Compatible does not mean identical. Parameters are compatible if
    they describe the same operation within acceptable variance.
  - See §4.2 for compatibility rules.

Phase 5: DOMAIN COVERAGE
  - For each domain affected by the converged position, did the
    relevant Counsel member consider it?
  - If a position affects ecology but the ecology Counsel did not
    evaluate, the convergence is incomplete — the decision was not
    examined from a relevant angle.

Phase 6: CONVERGENCE DECLARATION
  - If all phases pass: convergence has manifested.
  - Package the converged position into a ConvergenceDirective.
  - Record the directive hash on-chain via OracleFacet.
  - Route the directive back to the requesting agent via NIKO.
```

### 4.2 Parameter Compatibility

Parameters are not compared for strict equality. They are compared for operational equivalence — would executing with either set of parameters produce the same effect within constitutional bounds?

```typescript
/**
 * @title ParameterCompatibility
 * @notice Rules for determining if two parameter sets are compatible.
 * @dev Strict equality would make convergence unreachable —
 *      evaluators reason differently and will express the same
 *      intent with different specifics. The system checks
 *      whether the differences are material.
 */

/**
 * Compatibility rules by parameter type:
 *
 * Numeric values:
 *   Compatible if within ±10% of each other (configurable).
 *   Example: Agent A says allocate 1000, Agent B says allocate 950.
 *   These are compatible — the intent is the same.
 *
 * Enumerations / categorical values:
 *   Must be identical. No fuzzy matching on categories.
 *
 * String descriptions:
 *   Compared semantically, not literally. Two descriptions
 *   that describe the same operation in different words are compatible.
 *   Semantic comparison uses S.H.A.N.N.O.N.'s vector store for
 *   similarity scoring. Threshold: 0.85 cosine similarity (configurable).
 *
 * Domain lists:
 *   Compatible if they are set-equivalent (order does not matter).
 *   A superset is compatible with a subset if the additional domains
 *   do not introduce new risk.
 *
 * Resource estimates:
 *   Compatible if within the same order of magnitude and both
 *   fall within the agent's resource ceiling.
 */

interface CompatibilityConfig {
  /** Numeric tolerance (fraction, e.g., 0.10 = ±10%) */
  numericTolerance: number;

  /** Semantic similarity threshold for string comparison */
  semanticSimilarityThreshold: number;

  /** Whether superset domain lists are compatible with subsets */
  allowDomainSuperset: boolean;

  /** Resource estimate tolerance (fraction) */
  resourceEstimateTolerance: number;
}
```

### 4.3 Counsel Aggregation

The Counsel panel consists of multiple domain-weighted models. Each submits an independent evaluation. The Counsel's collective position is determined before comparison with Admin and Human.

```
Counsel domains: [biology, ecology, economics, infrastructure, ...]

For each active domain:
  - That domain's Counsel member submits an evaluation
  - The evaluation represents that domain's perspective on the decision

Counsel aggregation:
  1. Check that all active domains have submitted
  2. Check that all Counsel evaluations propose the same action type
  3. Check parameter compatibility across Counsel evaluations
  4. If Counsel internally converges: the Counsel position is the
     intersection of all domain evaluations
  5. If Counsel does NOT internally converge: this is itself a
     divergence point. The Counsel cannot present a unified position.
     Convergence cannot occur if Counsel is internally divided.

The Counsel's aggregated position is then compared with Admin and Human.
```

**Why Counsel must internally converge first:** If biology says "execute" and economics says "abort," the decision has not been examined from both angles and found coherent. It has been examined from both angles and found contradictory. Presenting this contradiction as a single Counsel position would be dishonest. The system surfaces the internal Counsel divergence as a divergence point rather than papering over it.

### 4.4 Divergence Analysis

When convergence does not occur, the detection algorithm produces a structured analysis of where evaluations diverge. This analysis serves two purposes: it feeds the Contentment Protocol (CONTENTMENT.md) so the system knows what logistical reality to address, and it feeds the Learning Substrate so S.H.A.N.N.O.N. can refine its constitutional field.

```typescript
/**
 * @title DivergenceAnalysis
 * @notice Structured analysis of why convergence did not occur.
 * @dev This is not a failure report. It is diagnostic information
 *      that directs the system's iterative mutualism response.
 *
 *      IMPORTANT: The detection algorithm (§4.1) is source-blind —
 *      it determines convergence or non-convergence without knowing
 *      which evaluator submitted which position. However, once
 *      non-convergence is established, the divergence analysis
 *      re-attaches evaluator identities for diagnostic purposes.
 *      The Contentment Protocol needs to know which evaluator is
 *      the constraint — e.g., "the Human did not respond" or
 *      "Counsel's ecology domain diverges from economics."
 *
 *      The blinded eye judges convergence. The sighted analysis
 *      diagnoses non-convergence. These are separate phases.
 */
interface DivergenceAnalysis {
  /** The convergence request that did not converge */
  requestId: string;

  /** Which phase of the detection algorithm failed */
  failedPhase: DetectionPhase;

  /** Specific points of divergence */
  divergencePoints: DivergencePoint[];

  /** Whether the divergence appears resolvable through additional information */
  informationGapDetected: boolean;

  /** Knowledge gaps identified across all evaluations */
  aggregatedKnowledgeGaps: string[];

  /** Recommended focus for iterative mutualism */
  suggestedFocus: string;
}

enum DetectionPhase {
  COMPLETENESS = 'completeness',
  CONSTITUTIONAL_CONSISTENCY = 'constitutional_consistency',
  ACTION_ALIGNMENT = 'action_alignment',
  PARAMETER_COMPATIBILITY = 'parameter_compatibility',
  DOMAIN_COVERAGE = 'domain_coverage',
}

interface DivergencePoint {
  /** Which evaluators diverge at this point */
  evaluators: EvaluatorId[];

  /** What they diverge on */
  topic: string;

  /** The nature of the divergence — not who is right, but where the gap is */
  description: string;

  /** The positions that diverge */
  positions: {
    evaluator: EvaluatorId;
    position: string;
  }[];

  /** Whether this divergence stems from different information
   *  (resolvable) or different values (may require constitutional evolution) */
  divergenceType: 'informational' | 'evaluative' | 'constitutional';
}
```

### 4.5 Divergence Types

Understanding the type of divergence determines the system's response.

| Type | Meaning | System Response |
|------|---------|----------------|
| **Informational** | Evaluators reached different conclusions because they had different information. One evaluator knew something the others did not. | Address the knowledge gap. The first logistical reality is the missing information. Agents research and surface the needed knowledge. Re-evaluation may converge. |
| **Evaluative** | Evaluators had the same information but weighed it differently. Different domains prioritize different factors. | This is the hardest divergence to resolve. The Contentment Protocol works on the underlying logistical constraint that makes the weighting trade-off necessary. If the constraint is removed, the trade-off dissolves. |
| **Constitutional** | Evaluators interpret a constitutional constraint differently. The decision space itself is ambiguous. | This divergence feeds directly to S.H.A.N.N.O.N.'s mutable cognitive layer. The constitutional field needs refinement. This is evolution in action — the system is discovering the edges of its own framework. |

---

## 5. Directive Generation

When convergence is detected, the system packages the manifested decision into a `ConvergenceDirective` and routes it back to the requesting agent.

### 5.1 Directive Construction

```typescript
/**
 * @title buildDirective
 * @notice Constructs a ConvergenceDirective from converged evaluations.
 * @dev The directive is the synthesis of all evaluations —
 *      not any single evaluation, but the intersection point.
 *
 * @param requestId Original convergence request
 * @param evaluations All submitted evaluations (verified converged)
 * @param constitutionalHash Current constitutional field hash
 * @returns The directive to route back to the agent
 */
function buildDirective(
  requestId: string,
  evaluations: Evaluation[],
  constitutionalHash: string,
): ConvergenceDirective {
  // Extract the converged action type (all evaluations agree on this)
  const actionType = evaluations[0].position.action;

  // Merge parameters — take the intersection of compatible params
  const mergedParameters = mergeCompatibleParameters(
    evaluations.map(e => e.parameters),
  );

  // Collect all domains considered across all evaluations
  const allDomains = new Set(
    evaluations.flatMap(e => e.domainsConsidered),
  );

  // Synthesize reasoning from all chains
  const synthesizedReasoning = synthesizeReasoning(
    evaluations.map(e => e.reasoning),
  );

  return {
    requestId,
    decision: formatDecisionDescription(actionType, mergedParameters),
    action: {
      actionType,
      parameters: mergedParameters,
      scopeUpdate: extractScopeUpdate(evaluations),
    },
    reasoning: synthesizedReasoning,
    counselDomains: evaluations
      .filter(e => e.evaluator.type === 'counsel')
      .map(e => (e.evaluator as { type: 'counsel'; domain: string }).domain),
    constitutionalHash,
    convergedAt: Date.now(),
  };
}
```

### 5.2 Directive Recording

Before the directive reaches the agent, it is recorded for provenance:

1. The full directive payload is stored in PostgreSQL.
2. A keccak256 hash of the directive is submitted to OracleFacet on the main chain.
3. The `brandi.convergence.achieved` event is published to the Event Bus.
4. The directive is sent to the requesting agent via the WebSocket channel `ws://niko/convergence/{requestId}/directive`.

### 5.3 Directive Immutability

Once a directive is issued, it cannot be modified or recalled. If the directive proves incorrect or the situation changes, a new convergence cycle must occur. The original directive and its outcome are preserved in the Learning Substrate as training data — including any negative outcomes. The system learns from its mistakes without concealing them.

---

## 6. Edge Cases

### 6.1 Constitutional Field Evolution During Evaluation

If S.H.A.N.N.O.N.'s constitutional field changes between the time evaluators receive the `DecisionContext` and the time they submit evaluations, the `constitutionalHash` values will differ. The detection algorithm catches this in Phase 2 (Constitutional Consistency).

**Response:** All evaluations are discarded. The convergence request is re-issued with the current constitutional hash. Evaluators re-evaluate in the updated field. This is not a failure — it is the system ensuring that all evaluators reason within the same framework.

### 6.2 Human Evaluator Unavailable

The Human may not respond within the evaluation window. This is the most common non-convergence scenario. The system does not treat this as a problem to solve by removing the Human from the loop.

**Response:** The convergence request reverts to iterative mutualism. The first logistical reality is the Human's capacity to participate. B.R.A.N.D.I. agents direct effort toward whatever is consuming the Human's bandwidth (see CONTENTMENT.md for the full reversion mechanics).

### 6.3 Single Counsel Domain

Early in the system's lifecycle, the Counsel panel may have only one active domain. The convergence model still requires three parties (Admin, Counsel, Human), but Counsel's internal aggregation is trivial with a single domain.

**Response:** The system operates normally. The single Counsel domain's evaluation IS the Counsel position. As the panel evolves and new domains are added, the aggregation becomes non-trivial. The system scales gracefully.

### 6.4 Identical Evaluations

If all evaluators submit structurally identical evaluations (same action, same parameters, same reasoning), convergence is trivially detected. This is not suspicious — it means the decision was obvious from all perspectives.

**Response:** Normal directive generation. The Learning Substrate records the unanimous convergence for pattern analysis.

### 6.5 Repeated Non-Convergence on the Same Decision

If the same decision repeatedly fails to converge after iterative mutualism cycles, the system does not force convergence. It does not escalate. It does not override.

**Response:** Each cycle generates training data. The divergence analysis becomes more refined with each attempt. If the divergence is constitutional, S.H.A.N.N.O.N.'s mutable cognitive layer evolves. If the divergence is evaluative, the underlying logistical constraint is progressively addressed. The system is patient. There is no deadline for convergence. There is only the continuing work of iterative mutualism.

---

## 7. Implementation Guidance

### 7.1 Build Order

```
1. Evaluation type definitions (Evaluation, DecisionPosition, ReasoningChain)
   └── 2. EvaluatorId type + evaluator routing
       └── 3. Evaluation sealed store (temporal isolation enforcement)
           └── 4. Detection algorithm phases 1-2 (completeness, constitutional consistency)
               └── 5. Detection algorithm phases 3-4 (action alignment, parameter compatibility)
                   └── 6. Counsel aggregation logic
                       └── 7. Detection algorithm phase 5 (domain coverage)
                           └── 8. Divergence analysis generation
                               └── 9. Directive construction + recording
                                   └── 10. Integration with NIKO event bus + WebSocket delivery
```

### 7.2 Testing Strategy

| Test | What to Verify |
|------|----------------|
| Temporal isolation | No evaluator can see another's evaluation before all are submitted |
| Completeness detection | Missing evaluations are correctly identified |
| Constitutional consistency | Hash mismatches are caught; re-evaluation is triggered |
| Action alignment | Divergent action types produce correct divergence points |
| Parameter compatibility | Numeric tolerance, semantic similarity, domain sets all work correctly |
| Counsel aggregation | Internal Counsel divergence is surfaced, not hidden |
| Divergence typing | Informational/evaluative/constitutional types are correctly classified |
| Directive construction | Merged parameters are correct. All domains are captured. Reasoning is synthesized. |
| Directive immutability | Issued directives cannot be modified after the fact |
| Window expiration | Missing evaluations revert to mutualism, not error state |
| Constitutional evolution | Mid-evaluation field change triggers re-evaluation, not failure |

---

**Copyright © 2024 John A. Welch, Director, Blinded Eye Foundation. All rights reserved.**
