# B.R.A.N.D.I. — Architecture Specification

**Copyright © 2024 John A. Welch, Director, Blinded Eye Foundation. All rights reserved.**

**Prerequisite:** Read `DESIGN.md` before this document. This spec translates that philosophy into technical architecture. Decisions here are subordinate to the principles established there.

---

## 1. System Topology

B.R.A.N.D.I. is a mid-stack system — it sits between S.H.A.N.N.O.N. (constitutional field) above and N.I.K.O.System (infrastructure) below. All three share a common infrastructure substrate but have distinct responsibilities and strict interface boundaries.

```
┌─────────────────────────────────────────────────────────────────┐
│                        S.H.A.N.N.O.N.                          │
│              Constitutional Field / Anti-Dysfunction            │
│                                                                 │
│  ┌──────────────┐  ┌────────────────┐  ┌─────────────────┐     │
│  │  Immutable   │  │    Mutable     │  │    Learning     │     │
│  │  Core        │  │  Cognitive     │  │   Substrate     │     │
│  │  (Axioms)    │  │  Layer (EIMs)  │  │  (Vector Store) │     │
│  └──────────────┘  └────────────────┘  └─────────────────┘     │
│                                                                 │
│  Exposes: Knowledge API, Report Ingestion API                   │
└────────────────────────────┬────────────────────────────────────┘
                             │
                    N.I.K.O.System Routes
                             │
┌────────────────────────────┼────────────────────────────────────┐
│                      B.R.A.N.D.I.                               │
│                  Agentic Automation Layer                        │
│                                                                 │
│  ┌────────────┐  ┌──────────────┐  ┌─────────────────────┐     │
│  │   Agent    │  │   Workflow   │  │    Convergence      │     │
│  │   Runtime  │  │   Engine     │  │    Module           │     │
│  └─────┬──────┘  └──────┬───────┘  └──────────┬──────────┘     │
│        │                │                      │                │
│        └────────────────┼──────────────────────┘                │
│                         │                                       │
│                  NIKO Integration Layer                          │
│            (single interface to infrastructure)                  │
└─────────────────────────┬───────────────────────────────────────┘
                          │
┌─────────────────────────┼───────────────────────────────────────┐
│                   N.I.K.O.System                                │
│              Infrastructure Substrate                            │
│                                                                 │
│  ┌───────────┐ ┌───────┐ ┌────────┐ ┌─────────┐ ┌──────────┐  │
│  │Orchestr-  │ │Kernel │ │Oracle  │ │Treasury │ │Monitoring│  │
│  │atorFacet  │ │Facet  │ │Facet   │ │Facet    │ │Facet     │  │
│  └───────────┘ └───────┘ └────────┘ └─────────┘ └──────────┘  │
│                                                                 │
│  ┌───────────┐ ┌───────┐ ┌────────┐ ┌──────────────────────┐   │
│  │PostgreSQL │ │Redis  │ │RxJS    │ │NestJS API Layer      │   │
│  │           │ │       │ │Event   │ │(REST/GraphQL/WS)     │   │
│  │           │ │       │ │Bus     │ │                      │   │
│  └───────────┘ └───────┘ └────────┘ └──────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Core Data Structures

These are the foundational types that flow through the system. All types are defined in TypeScript. On-chain representations are noted where applicable.

### 2.1 Agent Identity

```typescript
/**
 * @title AgentIdentity
 * @notice The complete identity of a B.R.A.N.D.I. agent.
 * @dev Maps to OrchestratorFacet.Agent on-chain.
 */
interface AgentIdentity {
  /** Unique agent identifier (bytes32 on-chain) */
  agentId: string;

  /** On-chain address for this agent */
  agentAddress: string;

  /** Agent type classification */
  agentType: AgentType;

  /** Human-readable name for logging and monitoring */
  name: string;

  /** IPFS hash or URI pointing to capability declaration */
  metadataUri: string;

  /** Declared capabilities this agent possesses */
  capabilities: AgentCapability[];

  /** Tools this agent has access to */
  tools: ToolDeclaration[];

  /** Scope boundaries — what this agent is authorized to do */
  scope: AgentScope;

  /** Current operational status */
  status: AgentStatus;

  /** Whether this agent is part of the active swarm */
  active: boolean;

  /** Timestamp of registration */
  registeredAt: number;

  /** Parent agent ID if spawned by a Spawner agent, null if root */
  parentAgentId: string | null;
}

enum AgentType {
  EXECUTOR = 'executor',
  CONVERSATIONAL = 'conversational',
  REACT = 'react',
  SPAWNER = 'spawner',
  AGGREGATOR = 'aggregator',
  SENTINEL = 'sentinel',
}

enum AgentStatus {
  IDLE = 0,
  RUNNING = 1,
  PAUSED = 2,
  ERROR = 3,
  INACTIVE = 4,
}
```

### 2.2 Agent Capability and Scope

```typescript
/**
 * @title AgentCapability
 * @notice A declared capability that an agent possesses.
 * @dev Used for boundary detection — agents can only act within
 *      their declared capabilities.
 */
interface AgentCapability {
  /** Unique identifier for this capability */
  capabilityId: string;

  /** Human-readable description */
  description: string;

  /** Domain this capability belongs to */
  domain: string;

  /** Whether this capability requires convergence for execution */
  requiresConvergence: boolean;
}

/**
 * @title AgentScope
 * @notice Defines the boundaries within which an agent operates.
 * @dev The agent's boundary detection system uses this to determine
 *      when to report-and-wait vs. act autonomously.
 */
interface AgentScope {
  /** Domains this agent is authorized to operate in */
  authorizedDomains: string[];

  /** Maximum resource expenditure before requiring convergence */
  resourceCeiling: ResourceLimit;

  /** Whether this agent can spawn sub-agents */
  canSpawn: boolean;

  /** Maximum depth of sub-agent spawning */
  maxSpawnDepth: number;

  /** Whether this agent can interact with humans directly */
  canInteractHuman: boolean;

  /** Action classifications that always require convergence */
  alwaysConverge: string[];
}

interface ResourceLimit {
  /** Maximum gas budget per action */
  maxGasPerAction: bigint;

  /** Maximum total gas budget per workflow */
  maxGasPerWorkflow: bigint;

  /** Maximum number of API calls per minute */
  maxApiCallsPerMinute: number;

  /** Maximum number of sub-agents that can be spawned */
  maxSubAgents: number;
}
```

### 2.3 Tool Declaration

```typescript
/**
 * @title ToolDeclaration
 * @notice Describes a tool available to an agent.
 * @dev Analogous to n8n nodes — each tool represents
 *      a capability to interact with an external system.
 */
interface ToolDeclaration {
  /** Unique tool identifier */
  toolId: string;

  /** Human-readable name */
  name: string;

  /** What this tool does */
  description: string;

  /** Input schema (JSON Schema format) */
  inputSchema: Record<string, unknown>;

  /** Output schema (JSON Schema format) */
  outputSchema: Record<string, unknown>;

  /** Whether this tool's actions require convergence */
  requiresConvergence: boolean;

  /** Whether this tool has side effects (writes, mutations, transactions) */
  hasSideEffects: boolean;

  /** Rate limits for this tool */
  rateLimit: { maxCalls: number; windowSeconds: number } | null;
}
```

### 2.4 Workflow Item

```typescript
/**
 * @title WorkflowItem
 * @notice The fundamental data unit flowing between nodes in a workflow.
 * @dev Extends n8n's JSON item model with provenance tracking
 *      and chain commitment metadata.
 */
interface WorkflowItem {
  /** The actual data payload (arbitrary JSON) */
  data: Record<string, unknown>;

  /** Provenance metadata — where this item came from */
  provenance: ItemProvenance;

  /** Whether this item requires on-chain recording */
  requiresChainCommitment: boolean;

  /** Hash of the constitutional field state when this item was created */
  constitutionalHash: string;
}

interface ItemProvenance {
  /** Agent that created this item */
  originAgentId: string;

  /** Workflow this item belongs to */
  workflowId: string;

  /** Node that produced this item */
  nodeId: string;

  /** Timestamp of creation */
  createdAt: number;

  /** Chain of transformations applied to this item */
  transformationChain: TransformationRecord[];
}

interface TransformationRecord {
  /** Node that performed the transformation */
  nodeId: string;

  /** Agent that operated the node */
  agentId: string;

  /** Type of transformation */
  transformationType: string;

  /** Timestamp */
  timestamp: number;

  /** Hash of the item before transformation (for auditability) */
  inputHash: string;
}
```

### 2.5 Convergence Request

```typescript
/**
 * @title ConvergenceRequest
 * @notice Submitted by an agent when a decision exceeds its scope.
 * @dev The agent packages all relevant context and submits
 *      through N.I.K.O.System to S.H.A.N.N.O.N. Then waits.
 */
interface ConvergenceRequest {
  /** Unique request identifier */
  requestId: string;

  /** Agent that initiated the request */
  originAgentId: string;

  /** Workflow context where the boundary was hit */
  workflowId: string;

  /** The decision that needs convergence */
  decision: DecisionContext;

  /** Current state of the agent when boundary was detected */
  agentState: Record<string, unknown>;

  /** Timestamp of request */
  requestedAt: number;

  /** Current status of this convergence request */
  status: ConvergenceStatus;
}

interface DecisionContext {
  /** Human-readable description of the decision */
  description: string;

  /** Why the agent determined this exceeds its scope */
  boundaryReason: string;

  /** Options the agent has identified (if any) */
  identifiedOptions: DecisionOption[];

  /** Relevant data the evaluating parties will need */
  supportingData: Record<string, unknown>;

  /** Domain(s) this decision affects */
  affectedDomains: string[];

  /** Estimated impact level */
  impactAssessment: ImpactLevel;
}

interface DecisionOption {
  /** Description of this option */
  description: string;

  /** Estimated consequences */
  projectedOutcome: string;

  /** Resource requirements */
  resourceCost: Record<string, unknown>;
}

enum ConvergenceStatus {
  /** Request submitted, awaiting evaluation */
  PENDING = 'pending',

  /** Admin/Counsel/Human are evaluating */
  EVALUATING = 'evaluating',

  /** Convergence achieved — directive ready */
  CONVERGED = 'converged',

  /** Non-convergence — reverted to iterative mutualism */
  REVERTED_TO_MUTUALISM = 'reverted_to_mutualism',
}

enum ImpactLevel {
  /** Affects only this agent's current task */
  LOCAL = 'local',

  /** Affects multiple agents or workflows */
  CLUSTER = 'cluster',

  /** Affects the swarm broadly */
  SWARM = 'swarm',

  /** Affects systems or participants outside B.R.A.N.D.I. */
  EXTERNAL = 'external',

  /** Affects the civilizational trajectory (Point B) */
  CIVILIZATIONAL = 'civilizational',
}
```

### 2.6 Convergence Directive

```typescript
/**
 * @title ConvergenceDirective
 * @notice The result of a successful convergence, returned to the agent.
 * @dev Flows back through N.I.K.O.System to the requesting agent.
 */
interface ConvergenceDirective {
  /** Matches the original ConvergenceRequest.requestId */
  requestId: string;

  /** The manifested decision */
  decision: string;

  /** Structured action the agent should execute */
  action: DirectiveAction;

  /** Reasoning summary (for the Learning Substrate) */
  reasoning: string;

  /** Which domains of the counsel panel contributed */
  counselDomains: string[];

  /** Constitutional hash at time of convergence */
  constitutionalHash: string;

  /** Timestamp of convergence */
  convergedAt: number;
}

interface DirectiveAction {
  /** Type of action the agent should take */
  actionType: 'execute' | 'delegate' | 'spawn' | 'abort' | 'modify_scope';

  /** Action-specific parameters */
  parameters: Record<string, unknown>;

  /** Updated scope if the agent's boundaries need adjustment */
  scopeUpdate: Partial<AgentScope> | null;
}
```

### 2.7 Agent Report

```typescript
/**
 * @title AgentReport
 * @notice Outcome report submitted by an agent after action completion.
 * @dev Routes through N.I.K.O.System to S.H.A.N.N.O.N.'s Learning Substrate.
 *      Also recorded on-chain via OrchestratorFacet.recordAgentAction().
 */
interface AgentReport {
  /** The agent submitting the report */
  agentId: string;

  /** Workflow context */
  workflowId: string;

  /** What was done */
  actionSummary: string;

  /** Hash of the action for on-chain recording */
  actionHash: string;

  /** Hash of the result for on-chain recording */
  resultHash: string;

  /** Gas consumed (for on-chain recording) */
  gasUsed: bigint;

  /** Whether this action resulted from a convergence directive */
  convergenceRequestId: string | null;

  /** Outcome classification */
  outcome: ReportOutcome;

  /** Timestamp */
  reportedAt: number;

  /** Any data that should feed the Learning Substrate */
  learningData: Record<string, unknown>;
}

enum ReportOutcome {
  SUCCESS = 'success',
  PARTIAL = 'partial',
  FAILURE = 'failure',
  BOUNDARY_HIT = 'boundary_hit',
}
```

### 2.8 Extended Type Definitions

The core data structures above define the foundational types that flow through the system. Domain-specific specifications extend these types with additional detail:

| Specification | Types Defined | Purpose |
|--------------|---------------|---------|
| `docs/CONVERGENCE.md` | `EvaluatorId`, `DecisionPosition`, `ReasoningChain`, `ReasoningStep`, `CompatibilityConfig`, `DivergenceAnalysis` (full), `DetectionPhase` | Convergence evaluation format, detection algorithm, divergence analysis |
| `docs/CONTENTMENT.md` | `DecompositionStep`, `ContentmentCycleRecord`, `MutualismOutcome`, `ActionRecord`, `KnowledgeContribution`, `IContentmentRegistry`, `SharedRealityGroup`, `ActiveLoop` | Iterative mutualism reversion, training data capture, concurrent loop management |
| `docs/INTEGRATION.md` | `IGasAccountant`, `IChainMapper`, `ChainMapping`, `AllocationThresholds`, `AllocationResult`, `AllocationRecord`, `CacheEntry` (expanded), `ComponentHealth`, `SystemEvent`, `EventSeverity` | N.I.K.O.System facet client operations, monitoring, resource allocation |

These types are authoritative in their respective specifications. This document defines the shared vocabulary; the specs define the domain-specific extensions.

---

## 3. Communication Architecture

All inter-component communication follows a single principle: **B.R.A.N.D.I. agents never communicate directly with S.H.A.N.N.O.N.** Everything routes through N.I.K.O.System.

### 3.1 Message Flow Patterns

**Pattern 1: Knowledge Request (Agent → SHANNON)**
```
Agent                    NIKO Backend            SHANNON
  │                          │                      │
  │── KnowledgeRequest ─────►│                      │
  │                          │── route to SHANNON ──►│
  │                          │                      │── query vector store
  │                          │                      │── apply constitutional
  │                          │                      │   curation
  │                          │◄── CuratedResponse ──│
  │◄── CuratedKnowledge ────│                      │
  │                          │                      │
```

**Pattern 2: Convergence (Agent → NIKO → SHANNON → Admin/Counsel/Human)**
```
Agent              NIKO              SHANNON           Convergence Module
  │                  │                  │                      │
  │── ConvergenceReq►│                  │                      │
  │                  │── record on-chain│                      │
  │                  │── route ────────►│                      │
  │                  │                  │── evaluate context ──►│
  │                  │                  │                      │── Admin evaluates
  │                  │                  │                      │── Counsel evaluates
  │                  │                  │                      │── Human evaluates
  │                  │                  │                      │
  │                  │                  │      ┌───────────────┤
  │                  │                  │      │CONVERGED?     │
  │                  │                  │      │               │
  │                  │                  │◄─YES─┤               │
  │                  │◄── Directive ───│      │               │
  │◄── Directive ───│                  │      │               │
  │                  │                  │  NO──┤               │
  │                  │                  │      │► Revert to    │
  │                  │                  │      │  Iterative    │
  │                  │                  │      │  Mutualism    │
  │                  │                  │      └───────────────┘
```

**Pattern 3: Report (Agent → NIKO → SHANNON)**
```
Agent                    NIKO Backend            SHANNON
  │                          │                      │
  │── AgentReport ──────────►│                      │
  │                          │── record on-chain    │
  │                          │   (OrchestratorFacet)│
  │                          │── route to SHANNON ──►│
  │                          │                      │── ingest to Learning
  │                          │                      │   Substrate
  │                          │◄── ack ─────────────│
  │◄── ack ────────────────│                      │
  │                          │                      │
```

### 3.2 Transport Layer

All B.R.A.N.D.I. ↔ N.I.K.O.System communication uses N.I.K.O.System's existing API infrastructure:

| Channel | Transport | Use Case |
|---------|-----------|----------|
| Agent lifecycle | REST API | Registration, status updates, deactivation |
| Real-time status | WebSocket | Live agent status, workflow progress |
| Knowledge queries | GraphQL | Structured queries to S.H.A.N.N.O.N. via NIKO |
| Convergence requests | REST + WebSocket | Submit via REST, await directive via WebSocket |
| Reports | REST API | Post-action outcome reporting |
| Chain events | RxJS Event Bus | Blockchain event listeners triggering workflows |
| Cache operations | Redis (via NIKO) | Hot state, intermediate results |

### 3.3 Event Types

Events flow through N.I.K.O.System's RxJS Event Bus. B.R.A.N.D.I. subscribes to relevant events and publishes its own.

```typescript
/**
 * @title BrandiEventType
 * @notice All event types published or consumed by B.R.A.N.D.I.
 */
enum BrandiEventType {
  // Agent lifecycle events
  AGENT_REGISTERED = 'brandi.agent.registered',
  AGENT_STATUS_CHANGED = 'brandi.agent.status_changed',
  AGENT_DEACTIVATED = 'brandi.agent.deactivated',
  AGENT_BOUNDARY_HIT = 'brandi.agent.boundary_hit',

  // Workflow events
  WORKFLOW_STARTED = 'brandi.workflow.started',
  WORKFLOW_COMPLETED = 'brandi.workflow.completed',
  WORKFLOW_FAILED = 'brandi.workflow.failed',
  WORKFLOW_BRANCHED = 'brandi.workflow.branched',
  WORKFLOW_CONSENSUS_BOUNDARY = 'brandi.workflow.consensus_boundary',

  // Convergence events
  CONVERGENCE_REQUESTED = 'brandi.convergence.requested',
  CONVERGENCE_ACHIEVED = 'brandi.convergence.achieved',
  CONVERGENCE_REVERTED = 'brandi.convergence.reverted',

  // Report events
  AGENT_REPORT_SUBMITTED = 'brandi.report.submitted',

  // Chain events (consumed from NIKO)
  CHAIN_BLOCK_FINALIZED = 'niko.chain.block_finalized',
  CHAIN_AGENT_ACTION_RECORDED = 'niko.chain.agent_action_recorded',
  CHAIN_AGGREGATION_COMPLETE = 'niko.chain.aggregation_complete',
}
```

---

## 4. Component Architecture

### 4.1 NIKO Integration Layer

This is B.R.A.N.D.I.'s single interface to infrastructure. All N.I.K.O.System access goes through this layer. No other B.R.A.N.D.I. component may access N.I.K.O.System directly.

```typescript
/**
 * @title INikoClient
 * @notice Interface for all N.I.K.O.System operations.
 * @dev Wraps OrchestratorFacet, KernelFacet, OracleFacet,
 *      MonitoringFacet, and TreasuryFacet into a single client.
 */
interface INikoClient {
  // Agent lifecycle (OrchestratorFacet)
  registerAgent(identity: AgentIdentity): Promise<void>;
  updateAgentStatus(agentId: string, status: AgentStatus): Promise<void>;
  deactivateAgent(agentId: string): Promise<void>;
  recordAgentAction(report: AgentReport): Promise<void>;
  getAgentMetrics(agentId: string): Promise<AgentMetrics>;

  // Caching (KernelFacet)
  cacheSet(key: string, valueHash: string, ttlSeconds: number): Promise<void>;
  cacheGet(key: string): Promise<CacheEntry | null>;
  cacheInvalidate(key: string): Promise<void>;

  // Chain operations (OracleFacet)
  submitResult(chainId: string, resultHash: string, proof: string): Promise<void>;
  aggregateResults(aggregationId: string, chainIds: string[], resultHashes: string[]): Promise<void>;

  // Monitoring (MonitoringFacet)
  reportHealth(component: string, health: ComponentHealth): Promise<void>;
  logEvent(event: SystemEvent): Promise<void>;

  // Treasury (TreasuryFacet)
  getBalance(address: string): Promise<bigint>;
  requestAllocation(amount: bigint, purpose: string): Promise<boolean>;

  // Routing to S.H.A.N.N.O.N.
  queryKnowledge(query: KnowledgeQuery): Promise<CuratedKnowledge>;
  submitConvergenceRequest(request: ConvergenceRequest): Promise<string>;
  submitReport(report: AgentReport): Promise<void>;
  awaitDirective(requestId: string): Promise<ConvergenceDirective>;
}
```

### 4.2 SHANNON Integration Layer

B.R.A.N.D.I. does not talk to S.H.A.N.N.O.N. directly. This layer defines the message formats that N.I.K.O.System carries between them.

```typescript
/**
 * @title KnowledgeQuery
 * @notice Request for constitutionally curated knowledge.
 */
interface KnowledgeQuery {
  /** The agent requesting knowledge */
  agentId: string;

  /** Natural language query */
  query: string;

  /** Domain context to focus the retrieval */
  domain: string;

  /** Maximum number of results */
  maxResults: number;
}

/**
 * @title CuratedKnowledge
 * @notice Response from S.H.A.N.N.O.N. — knowledge already vetted
 *         against constitutional axioms.
 */
interface CuratedKnowledge {
  /** The original query */
  queryId: string;

  /** Retrieved knowledge chunks */
  chunks: KnowledgeChunk[];

  /** Constitutional hash at time of curation */
  constitutionalHash: string;

  /** Timestamp */
  retrievedAt: number;
}

interface KnowledgeChunk {
  /** Content of the knowledge chunk */
  content: string;

  /** Source metadata */
  source: string;

  /** Relevance score from vector similarity */
  relevanceScore: number;

  /** Domain classification */
  domain: string;
}
```

### 4.3 Agent Runtime

The core agent execution environment. Every agent type shares this runtime.

```typescript
/**
 * @title IAgentRuntime
 * @notice The runtime environment every B.R.A.N.D.I. agent operates within.
 * @dev Implements the single-loop operating model described in DESIGN.md.
 *      The agent has one mode: operate within the constitutional field
 *      through iterative mutualism. Scale of engagement changes,
 *      but operating principle does not.
 */
interface IAgentRuntime {
  /** The agent's identity */
  readonly identity: AgentIdentity;

  /** Access to N.I.K.O.System (and through it, S.H.A.N.N.O.N.) */
  readonly niko: INikoClient;

  /** Execute the agent's primary loop */
  run(input: WorkflowItem[]): Promise<WorkflowItem[]>;

  /** Check if an action exceeds the agent's scope */
  detectBoundary(action: ProposedAction): BoundaryCheckResult;

  /** Submit a convergence request and wait for directive */
  requestConvergence(decision: DecisionContext): Promise<ConvergenceDirective>;

  /** Report an action outcome */
  report(outcome: AgentReport): Promise<void>;

  /** Query S.H.A.N.N.O.N. for knowledge (via NIKO) */
  queryKnowledge(query: string, domain: string): Promise<CuratedKnowledge>;

  /** Graceful shutdown */
  shutdown(): Promise<void>;
}

interface ProposedAction {
  /** What the agent intends to do */
  description: string;

  /** Which tools would be used */
  toolIds: string[];

  /** Estimated resource cost */
  estimatedCost: Partial<ResourceLimit>;

  /** Domains affected */
  affectedDomains: string[];

  /** Does this action have side effects? */
  hasSideEffects: boolean;
}

interface BoundaryCheckResult {
  /** Is this action within scope? */
  withinScope: boolean;

  /** If not, why? */
  reason: string | null;

  /** Recommended action */
  recommendation: 'proceed' | 'request_convergence';
}
```

### 4.4 Workflow Engine

```typescript
/**
 * @title IWorkflowEngine
 * @notice Orchestrates the execution of workflows composed of nodes.
 * @dev Extends n8n's paradigm with recursion, branching,
 *      and consensus boundaries.
 */
interface IWorkflowEngine {
  /** Create and register a new workflow */
  createWorkflow(definition: WorkflowDefinition): Promise<string>;

  /** Execute a workflow */
  executeWorkflow(workflowId: string, trigger: TriggerEvent): Promise<WorkflowResult>;

  /** Get workflow status */
  getWorkflowStatus(workflowId: string): Promise<WorkflowStatus>;

  /** Cancel a running workflow */
  cancelWorkflow(workflowId: string): Promise<void>;
}

interface WorkflowDefinition {
  /** Unique workflow identifier */
  workflowId: string;

  /** Human-readable name */
  name: string;

  /** Description of what this workflow does */
  description: string;

  /** Nodes in this workflow */
  nodes: WorkflowNode[];

  /** Connections between nodes */
  connections: NodeConnection[];

  /** Trigger configuration */
  trigger: TriggerConfig;

  /** Recursion configuration (if this workflow is recursive) */
  recursion: RecursionConfig | null;

  /** Resource limits for this workflow */
  resourceLimits: ResourceLimit;
}

interface WorkflowNode {
  /** Unique node identifier within this workflow */
  nodeId: string;

  /** Node type */
  nodeType: NodeType;

  /** Agent type that should execute this node */
  agentType: AgentType;

  /** Node-specific configuration */
  config: Record<string, unknown>;

  /** Whether this node is a consensus boundary */
  isConsensusBoundary: boolean;

  /** Whether this node is a branch point */
  isBranchPoint: boolean;
}

/**
 * @title NodeType
 * @notice All workflow node types.
 * @dev See docs/WORKFLOWS.md §2.1 for authoritative node type
 *      definitions, execution semantics, and agent type assignments.
 */
enum NodeType {
  /** Perform a tool action */
  ACTION = 'action',

  /** Transform data */
  TRANSFORM = 'transform',

  /** Conditional branch */
  CONDITION = 'condition',

  /** Merge parallel branches */
  MERGE = 'merge',

  /** Consensus boundary — all parallel branches must sync */
  CONSENSUS_BOUNDARY = 'consensus_boundary',

  /** Fork into parallel branches */
  BRANCH = 'branch',

  /** Request convergence */
  CONVERGENCE_GATE = 'convergence_gate',

  /** Report to Learning Substrate */
  REPORT = 'report',

  /** Marks the start of a recursive loop */
  RECURSION_ENTRY = 'recursion_entry',

  /** Evaluates termination condition for recursive loops */
  RECURSION_EXIT = 'recursion_exit',

  /** Query S.H.A.N.N.O.N. for curated knowledge via NIKO */
  KNOWLEDGE_QUERY = 'knowledge_query',

  /** Submit result hash to N.I.K.O.System's OracleFacet */
  CHAIN_COMMIT = 'chain_commit',
}

interface NodeConnection {
  /** Source node */
  fromNodeId: string;

  /** Target node */
  toNodeId: string;

  /** Output index (for nodes with multiple outputs) */
  outputIndex: number;

  /** Input index (for nodes with multiple inputs) */
  inputIndex: number;

  /** Condition for this connection (for conditional branches) */
  condition: string | null;
}

interface TriggerConfig {
  /** Trigger type */
  type: TriggerType;

  /** Type-specific configuration */
  config: Record<string, unknown>;
}

enum TriggerType {
  /** External HTTP request */
  WEBHOOK = 'webhook',

  /** Time-based schedule */
  SCHEDULE = 'schedule',

  /** On-chain event from N.I.K.O.System */
  CHAIN_EVENT = 'chain_event',

  /** Another agent's action or report */
  AGENT_EVENT = 'agent_event',

  /** A convergence directive was received */
  CONVERGENCE_EVENT = 'convergence_event',

  /** Manual trigger */
  MANUAL = 'manual',
}

interface RecursionConfig {
  /** Maximum number of recursive iterations */
  maxIterations: number;

  /** Condition to terminate recursion */
  terminationCondition: string;

  /** How to feed output back as input */
  feedbackMapping: Record<string, string>;

  /** Convergence checkpoint interval (check every N iterations) */
  convergenceCheckInterval: number;
}
```

### 4.5 Convergence Module

> **Detailed specification:** `docs/CONVERGENCE.md` is authoritative for evaluation format, detection algorithm, parameter compatibility, divergence analysis, and directive construction. The interfaces below define the module's API surface. `docs/CONTENTMENT.md` is authoritative for iterative mutualism reversion mechanics, first logistical reality identification, and training data capture.

> **The Blinded Eye:** Convergence detection is source-blind. On-chain, each evaluator is individually identified for auditability and provenance. But the detection algorithm itself does not see who submitted an evaluation — it sees positions, reasoning chains, and parameters. This is a constitutional constraint, not a convenience. The Foundation name is not decorative. Convergence is the blinded eye: it judges the reasoning, never the reasoner.

```typescript
/**
 * @title IConvergenceModule
 * @notice Manages the convergence process.
 * @dev This module is NOT called by agents directly.
 *      Agents submit requests through the NIKO integration layer.
 *      This module is invoked by the system when a convergence
 *      request arrives from NIKO/SHANNON.
 *
 *      Evaluation submission is unified — a single method accepts
 *      evaluations from Admin, Counsel, and Human identically.
 *      The evaluator identity is recorded on-chain for provenance
 *      but is NOT visible to the detection algorithm. The detection
 *      algorithm operates on positions and reasoning, blind to source.
 *
 *      See CONVERGENCE.md for full Evaluation, DecisionPosition,
 *      ReasoningChain, EvaluatorId, CompatibilityConfig, and
 *      DivergenceAnalysis type definitions.
 */
interface IConvergenceModule {
  /** Initialize a convergence process */
  initiate(request: ConvergenceRequest): Promise<void>;

  /**
   * Submit an evaluation from any evaluating party.
   * The evaluator identity within the Evaluation is recorded
   * on-chain for provenance but stripped before the evaluation
   * reaches the detection algorithm.
   */
  submitEvaluation(requestId: string, evaluation: Evaluation): Promise<void>;

  /** Check for convergence across all evaluations */
  checkConvergence(requestId: string): ConvergenceCheckResult;

  /** Revert to iterative mutualism when convergence fails */
  revertToMutualism(requestId: string): Promise<MutualismContext>;
}

/**
 * @title Evaluation
 * @notice Abbreviated interface — see CONVERGENCE.md §3 for the
 *         full specification including DecisionPosition, ReasoningChain,
 *         and EvaluatorId types.
 * @dev All evaluators use the same structure. The detection algorithm
 *      operates on position and reasoning, blind to evaluator identity.
 */
interface Evaluation {
  /** Evaluator identity — recorded on-chain, stripped before detection */
  evaluator: EvaluatorId;

  /** Hash of the decision being evaluated */
  decisionHash: string;

  /** The evaluator's position in the decision space */
  position: DecisionPosition;

  /** Structured reasoning chain */
  reasoning: ReasoningChain;

  /** Confidence level (0-1) — metadata, not a weight */
  confidence: number;

  /** Domains considered in this evaluation */
  domainsConsidered: string[];

  /** Constitutional hash at time of evaluation */
  constitutionalHash: string;

  /** Timestamp */
  evaluatedAt: number;
}

interface ConvergenceCheckResult {
  /** Did convergence occur? */
  converged: boolean;

  /** If converged, the manifested decision */
  directive: ConvergenceDirective | null;

  /** If not converged, structured divergence analysis */
  divergenceAnalysis: DivergenceAnalysis | null;
}

/**
 * @title DivergenceAnalysis
 * @notice Abbreviated — see CONVERGENCE.md §4.4 for the full
 *         specification including DetectionPhase and divergence typing.
 */
interface DivergenceAnalysis {
  /** Which phase of detection failed */
  failedPhase: string;

  /** Specific divergence points */
  divergencePoints: DivergencePoint[];

  /** Whether an information gap was detected */
  informationGapDetected: boolean;

  /** Recommended focus for iterative mutualism */
  suggestedFocus: string;
}

interface DivergencePoint {
  /** What the divergence is about — NOT who diverges */
  topic: string;

  /** The nature of the divergence */
  description: string;

  /** Whether this is informational, evaluative, or constitutional */
  divergenceType: 'informational' | 'evaluative' | 'constitutional';
}

interface MutualismContext {
  /** The original convergence request */
  requestId: string;

  /** Identified first logistical reality */
  firstLogisticalReality: LogisticalReality;

  /** Recommended agent actions to address the constraint */
  recommendedActions: DirectiveAction[];
}

interface LogisticalReality {
  /** Description of the constraint */
  description: string;

  /** Scale of the constraint */
  scale: 'personal' | 'physical' | 'civilizational';

  /** Domain(s) affected */
  domains: string[];

  /** Is this constraint addressable by B.R.A.N.D.I. agents? */
  agentAddressable: boolean;

  /** Estimated effort to address */
  estimatedEffort: string;
}
```

---

## 5. State Management

### 5.1 State Distribution

| State Type | Storage | Rationale |
|-----------|---------|-----------|
| Agent identity & lifecycle | On-chain (OrchestratorFacet) | Tamper-proof provenance |
| Agent hot state (current task, context) | Redis (via NIKO) | Fast access, ephemeral |
| Workflow definitions | PostgreSQL (via NIKO) | Persistent, queryable |
| Workflow execution state | PostgreSQL + Redis | Persistent with hot cache |
| Convergence requests & directives | PostgreSQL + On-chain hash | Persistent with provenance |
| Agent reports | PostgreSQL → S.H.A.N.N.O.N. Learning Substrate + On-chain hash | Long-term learning + provenance |
| Cached intermediate results | Redis (via KernelFacet) | TTL-managed, fast |
| Constitutional field state | S.H.A.N.N.O.N. (via NIKO) | Authoritative source |

### 5.2 On-Chain vs. Off-Chain

**On-chain (immutable provenance):**
- Agent registration and deactivation
- Action hashes and result hashes
- Convergence directive hashes
- Constitutional field hashes (for auditability)
- Resource allocation transactions

**Off-chain (operational state):**
- Full agent context and working memory
- Workflow execution progress
- Complete convergence request/response payloads
- Knowledge query results
- Full report content

The on-chain layer provides *proof that something happened*. The off-chain layer provides *the full detail of what happened*. They are linked by hashes.

---

## 6. Security Model

### 6.1 Role-Based Access (via LibSecurity)

| Role | Holder | Permissions |
|------|--------|------------|
| AGENT_MANAGER_ROLE | B.R.A.N.D.I. system process | Register/deactivate agents, update status |
| KERNEL_OPERATOR_ROLE | B.R.A.N.D.I. system process | Cache operations |
| ORACLE_OPERATOR_ROLE | B.R.A.N.D.I. system process | Submit/aggregate chain results |
| MONITORING_ROLE | Sentinel agents | Report health, log events |

Individual agents do NOT hold roles directly. The B.R.A.N.D.I. system process acts on behalf of agents, enforcing scope boundaries before passing requests to N.I.K.O.System.

### 6.2 Agent Isolation

Agents are isolated by scope. An agent cannot:
- Access another agent's working memory.
- Execute tools outside its declared tool set.
- Act in domains outside its authorized domains.
- Exceed its resource ceilings.
- Spawn sub-agents beyond its max spawn depth.

The Agent Runtime enforces these constraints before any action reaches N.I.K.O.System.

### 6.3 Convergence Integrity

The convergence model's hard-coded constraint (no execution without convergence for high-level decisions) is enforced at the smart contract level. The specific mechanism:

1. High-impact actions are flagged with `requiresConvergence: true`.
2. The NIKO integration layer checks for a valid `ConvergenceDirective` before executing.
3. The on-chain record of the directive hash is verified against the off-chain directive content.
4. If the hashes don't match, execution is rejected.

---

## 7. Performance Targets

Inherited from N.I.K.O.System's proven benchmarks and extended for B.R.A.N.D.I.:

| Metric | Target | Basis |
|--------|--------|-------|
| Concurrent agents | 10,000+ | OrchestratorFacet stress tests |
| Cache operations | 11.1M+ ops/sec | KernelFacet benchmarks |
| Blockchain throughput | 454,000+ blocks/sec | Dual-chain architecture |
| Agent registration gas | < 200,000 gas | Phase 1 test suite |
| Workflow engine latency | < 100ms per node execution | Target |
| Knowledge query latency | < 500ms | Target (includes vector search) |
| Convergence detection | < 1s after all evaluations received | Target |

---

**Copyright © 2024 John A. Welch, Director, Blinded Eye Foundation. All rights reserved.**
