# B.R.A.N.D.I. — Workflow Specification

**Copyright © 2024 John A. Welch, Director, Blinded Eye Foundation. All rights reserved.**

**Prerequisites:** Read `DESIGN.md`, `docs/ARCHITECTURE.md`, and `docs/AGENTS.md` before this document.

---

## 1. What a Workflow Is

A B.R.A.N.D.I. workflow is a directed graph of operations that agents execute. It is the unit of coordinated work — the thing that turns individual agent actions into collective outcomes.

B.R.A.N.D.I. workflows inherit the foundational paradigm from n8n (nodes → connections → workflows) and extend it with three capabilities n8n does not have:

- **Recursive pipelines** — workflows that feed their output back as input
- **Branching execution** — workflows that fork into parallel paths with independent agent clusters
- **Consensus boundaries** — synchronization points where parallel paths must align before proceeding

Every workflow operates within S.H.A.N.N.O.N.'s constitutional field. Every data item flowing through a workflow carries provenance metadata. Every significant result can be committed on-chain through N.I.K.O.System.

---

## 2. Workflow Primitives

### 2.1 Nodes

A node is a single unit of work within a workflow. Each node is executed by an agent.

```typescript
/**
 * @title WorkflowNode
 * @notice A single operation in a workflow graph.
 * @dev Each node is assigned to an agent type for execution.
 *      The workflow engine is responsible for routing items
 *      to the correct agent.
 */
interface WorkflowNode {
  /** Unique identifier within this workflow */
  nodeId: string;

  /** Human-readable label (for monitoring and debugging) */
  label: string;

  /** What kind of operation this node performs */
  nodeType: NodeType;

  /** Which agent type should execute this node */
  agentType: AgentType;

  /** Node-specific configuration */
  config: Record<string, unknown>;

  /** Whether this node is a consensus boundary */
  isConsensusBoundary: boolean;

  /** Whether this node is a branch point (forks into parallel paths) */
  isBranchPoint: boolean;

  /** Whether results from this node require on-chain commitment */
  requiresChainCommitment: boolean;

  /** Position in the visual editor (for UI rendering, no execution effect) */
  position: { x: number; y: number };
}
```

**Node types and their purposes:**

| NodeType | Purpose | Executed By |
|----------|---------|------------|
| `ACTION` | Perform a tool operation (API call, data write, external interaction) | Executor Agent |
| `TRANSFORM` | Modify, filter, reshape, or enrich data items | Executor Agent |
| `CONDITION` | Evaluate a condition and route items to different outputs | Executor or ReAct Agent |
| `BRANCH` | Fork execution into parallel paths, spawning agent clusters | Spawner Agent |
| `MERGE` | Combine items from multiple inputs into a single stream | Aggregator Agent |
| `CONSENSUS_BOUNDARY` | Synchronize parallel branches — all must complete before proceeding | Aggregator Agent |
| `CONVERGENCE_GATE` | Halt execution and submit a convergence request. Resume on directive. | Any agent (triggers report-and-wait) |
| `REPORT` | Submit data to S.H.A.N.N.O.N.'s Learning Substrate via NIKO | Any agent |
| `RECURSION_ENTRY` | Marks the start of a recursive loop | ReAct Agent |
| `RECURSION_EXIT` | Evaluates termination condition. Routes to exit or back to entry. | ReAct Agent |
| `KNOWLEDGE_QUERY` | Query S.H.A.N.N.O.N. for curated knowledge via NIKO | Any agent |
| `CHAIN_COMMIT` | Submit result hash to N.I.K.O.System's OracleFacet | Executor Agent |

### 2.2 Connections

A connection defines how data flows from one node to another.

```typescript
/**
 * @title NodeConnection
 * @notice A directed edge in the workflow graph.
 * @dev Data flows from fromNodeId's output to toNodeId's input.
 *      Connections can be conditional — only items matching the
 *      condition are routed through.
 */
interface NodeConnection {
  /** Unique connection identifier */
  connectionId: string;

  /** Source node */
  fromNodeId: string;

  /** Target node */
  toNodeId: string;

  /** Which output of the source node (nodes can have multiple outputs) */
  outputIndex: number;

  /** Which input of the target node (nodes can have multiple inputs) */
  inputIndex: number;

  /**
   * Optional condition for this connection.
   * If set, only items where the condition evaluates to true are routed.
   * Used by CONDITION nodes to implement branching logic.
   * Expression format: JSONPath or simple JS expression.
   */
  condition: string | null;

  /**
   * Data mapping — optional transformation applied during transit.
   * Maps fields from the source output schema to the target input schema.
   */
  dataMapping: Record<string, string> | null;
}
```

### 2.3 Workflows

A workflow is the complete graph plus its trigger, resource limits, and recursion configuration.

```typescript
/**
 * @title WorkflowDefinition
 * @notice Complete specification of an executable workflow.
 * @dev Stored in PostgreSQL via NIKO backend.
 *      The workflow engine interprets this definition at runtime.
 */
interface WorkflowDefinition {
  /** Unique workflow identifier */
  workflowId: string;

  /** Semantic version of this workflow definition */
  version: string;

  /** Human-readable name */
  name: string;

  /** What this workflow accomplishes */
  description: string;

  /** All nodes in this workflow */
  nodes: WorkflowNode[];

  /** All connections between nodes */
  connections: NodeConnection[];

  /** What triggers this workflow */
  trigger: TriggerConfig;

  /** Recursion configuration (null if not recursive) */
  recursion: RecursionConfig | null;

  /** Resource limits for the entire workflow execution */
  resourceLimits: ResourceLimit;

  /** Constitutional hash at time of workflow creation */
  constitutionalHash: string;

  /** Whether this workflow is active (can be triggered) */
  active: boolean;

  /** Creation metadata */
  createdBy: string;
  createdAt: number;
  updatedAt: number;
}
```

---

## 3. Triggers

A trigger is the event that initiates a workflow execution. Nothing happens until a trigger fires.

### 3.1 Trigger Types

```typescript
/**
 * @title TriggerConfig
 * @notice Defines what initiates a workflow.
 */
interface TriggerConfig {
  /** Trigger type */
  type: TriggerType;

  /** Type-specific configuration */
  config: WebhookTriggerConfig | ScheduleTriggerConfig | ChainEventTriggerConfig |
          AgentEventTriggerConfig | ConvergenceEventTriggerConfig | ManualTriggerConfig;
}
```

**WEBHOOK — External HTTP request**

An external system sends a request to a unique URL. The workflow starts with the request payload as its first `WorkflowItem[]`.

```typescript
interface WebhookTriggerConfig {
  /** HTTP method to listen for */
  method: 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE';

  /** Custom path suffix for the webhook URL */
  path: string;

  /** Authentication method */
  auth: 'none' | 'basic' | 'header' | 'jwt';

  /** Auth configuration (credentials stored securely, never in workflow definition) */
  authConfig: Record<string, unknown>;

  /** Response mode */
  responseMode: 'immediate' | 'when_complete' | 'streaming';
}
```

**SCHEDULE — Time-based**

Workflow runs at configured intervals.

```typescript
interface ScheduleTriggerConfig {
  /** Cron expression */
  cron: string;

  /** Timezone */
  timezone: string;

  /** Whether to catch up on missed executions */
  catchUpOnMissed: boolean;
}
```

**CHAIN_EVENT — On-chain event from N.I.K.O.System**

A blockchain event (agent registered, action recorded, aggregation complete, etc.) triggers the workflow.

```typescript
interface ChainEventTriggerConfig {
  /** Which event types to listen for */
  eventTypes: string[];

  /** Optional filter on event data */
  filter: Record<string, unknown> | null;

  /** Which chain to listen on (private client chain ID or 'main') */
  chainId: string;
}
```

**AGENT_EVENT — Another agent's action or report**

One agent's output triggers another workflow.

```typescript
interface AgentEventTriggerConfig {
  /** Agent ID to watch */
  sourceAgentId: string;

  /** Event type from that agent */
  eventType: BrandiEventType;

  /** Optional filter on event data */
  filter: Record<string, unknown> | null;
}
```

**CONVERGENCE_EVENT — A decision has manifested**

A convergence directive triggers execution of the decided action.

```typescript
interface ConvergenceEventTriggerConfig {
  /** Convergence request ID to watch for */
  convergenceRequestId: string;

  /** Or: watch for any convergence in specified domains */
  watchDomains: string[] | null;
}
```

**MANUAL — Human or system-initiated**

Explicitly started by a human or system process. No automated trigger.

```typescript
interface ManualTriggerConfig {
  /** Input schema the manual trigger expects */
  inputSchema: Record<string, unknown>;

  /** Who is authorized to manually trigger */
  authorizedTriggers: string[];
}
```

### 3.2 Trigger-to-Item Mapping

Every trigger produces a `WorkflowItem[]` as its output. The workflow engine normalizes trigger data into the standard item format before passing it to the first node.

```typescript
/**
 * @notice Trigger output normalization.
 * @dev Regardless of trigger type, the first node always receives
 *      WorkflowItem[] with provenance showing the trigger as origin.
 */
function normalizeTriggerOutput(
  trigger: TriggerConfig,
  rawData: unknown,
  workflowId: string,
): WorkflowItem[] {
  return [{
    data: rawData as Record<string, unknown>,
    provenance: {
      originAgentId: 'system:trigger',
      workflowId,
      nodeId: 'trigger',
      createdAt: Date.now(),
      transformationChain: [],
    },
    requiresChainCommitment: false,
    constitutionalHash: getCurrentConstitutionalHash(),
  }];
}
```

---

## 4. Execution Model

### 4.1 How the Workflow Engine Runs a Workflow

```
    Trigger fires
         │
         ▼
    Create WorkflowExecution record
    (status: RUNNING, store in PostgreSQL)
         │
         ▼
    Normalize trigger data → WorkflowItem[]
         │
         ▼
    Route items to first node(s)
         │
         ▼
    ┌─────────────────────────────────────────────┐
    │            NODE EXECUTION LOOP              │
    │                                             │
    │  For each node with pending input:          │
    │                                             │
    │  1. Select agent (by agentType)             │
    │  2. Pass WorkflowItem[] to agent.run()      │
    │  3. Agent executes (single-loop model)      │
    │  4. Agent returns WorkflowItem[] output      │
    │  5. Attach provenance to output items       │
    │  6. If requiresChainCommitment: submit hash │
    │  7. Route output to connected nodes         │
    │  8. If node is CONSENSUS_BOUNDARY: wait     │
    │     for all parallel inputs                 │
    │  9. If node is CONVERGENCE_GATE: submit     │
    │     convergence request, pause this path    │
    │ 10. If node is RECURSION_EXIT: evaluate     │
    │     termination condition                   │
    │                                             │
    │  Continue until no nodes have pending input │
    └─────────────────────────────────────────────┘
         │
         ▼
    Update WorkflowExecution record
    (status: COMPLETED or FAILED)
         │
         ▼
    Submit final report to Learning Substrate
```

### 4.2 Workflow Execution Record

```typescript
/**
 * @title WorkflowExecution
 * @notice Runtime state of a single workflow execution.
 * @dev Stored in PostgreSQL. Hot state cached in Redis.
 */
interface WorkflowExecution {
  /** Unique execution identifier */
  executionId: string;

  /** Which workflow definition is being executed */
  workflowId: string;

  /** Current status */
  status: WorkflowExecutionStatus;

  /** What triggered this execution */
  triggeredBy: TriggerType;

  /** Trigger data (normalized) */
  triggerData: WorkflowItem[];

  /** Per-node execution state */
  nodeStates: Map<string, NodeExecutionState>;

  /** Active branch tracking */
  branches: BranchState[];

  /** Recursion state (if applicable) */
  recursionState: RecursionState | null;

  /** Pending convergence requests */
  pendingConvergences: string[];

  /** Resource usage so far */
  resourceUsage: ResourceUsage;

  /** Timing */
  startedAt: number;
  completedAt: number | null;

  /** Final output (when complete) */
  output: WorkflowItem[] | null;

  /** Error details (if failed) */
  error: WorkflowError | null;
}

enum WorkflowExecutionStatus {
  /** Currently executing */
  RUNNING = 'running',

  /** Waiting on convergence */
  AWAITING_CONVERGENCE = 'awaiting_convergence',

  /** Completed successfully */
  COMPLETED = 'completed',

  /** Failed with error */
  FAILED = 'failed',

  /** Cancelled by user or system */
  CANCELLED = 'cancelled',

  /** Reverted to iterative mutualism */
  REVERTED_TO_MUTUALISM = 'reverted_to_mutualism',
}

interface NodeExecutionState {
  /** Node identifier */
  nodeId: string;

  /** Current state of this node */
  status: 'pending' | 'running' | 'completed' | 'failed' | 'waiting' | 'skipped';

  /** Input items received */
  inputItems: WorkflowItem[];

  /** Output items produced */
  outputItems: WorkflowItem[];

  /** Agent that executed this node */
  executingAgentId: string | null;

  /** Timing */
  startedAt: number | null;
  completedAt: number | null;

  /** Error if failed */
  error: string | null;
}

interface ResourceUsage {
  /** Total gas consumed across all on-chain operations */
  totalGas: bigint;

  /** Total API calls made */
  totalApiCalls: number;

  /** Total agents spawned */
  totalAgentsSpawned: number;

  /** Total knowledge queries */
  totalKnowledgeQueries: number;

  /** Wall clock time elapsed (ms) */
  elapsedMs: number;
}
```

### 4.3 Agent Assignment

The workflow engine assigns agents to nodes based on `agentType`. The engine does not create agents — it requests them from the agent pool or asks a Spawner to create them.

```typescript
/**
 * @title Agent assignment strategy
 * @dev The engine maintains an agent pool. When a node needs execution:
 *
 * 1. Check if a suitable agent (matching type, sufficient scope) is IDLE
 * 2. If yes: assign the task to that agent
 * 3. If no IDLE agent available:
 *    a. If a Spawner is available: request a new agent
 *    b. If no Spawner: queue the task until an agent becomes available
 * 4. Never exceed workflow resource limits when spawning
 */
interface AgentAssignment {
  /** The node being executed */
  nodeId: string;

  /** The assigned agent */
  agentId: string;

  /** Assignment timestamp */
  assignedAt: number;

  /** Expected completion (for timeout detection by Sentinels) */
  expectedCompletionAt: number | null;
}
```

---

## 5. Recursive Pipelines

A recursive workflow feeds its output back as its own input, refining results through iterative cycles. This is how B.R.A.N.D.I. implements the "Recursive" in its name.

### 5.1 Recursion Structure

```
    ┌─────────────────────────────────────┐
    │                                     │
    │     RECURSION_ENTRY                 │
    │     (receives initial input         │
    │      OR feedback from prior cycle)  │
    │                                     │
    └──────────────┬──────────────────────┘
                   │
                   ▼
          Processing nodes
          (ACTION, TRANSFORM, etc.)
                   │
                   ▼
    ┌──────────────────────────────────────┐
    │     RECURSION_EXIT                   │
    │                                      │
    │  Evaluate termination condition:     │
    │                                      │
    │  1. Has quality threshold been met?  │
    │  2. Has max iteration been reached?  │
    │  3. Has resource limit been hit?     │
    │  4. Has convergence check interval   │
    │     been reached? (verify still      │
    │     within constitutional space)     │
    │                                      │
    └────────┬───────────────────┬─────────┘
             │                   │
          CONTINUE             TERMINATE
             │                   │
             ▼                   ▼
        Map output to       Output final
        input via            result to
        feedbackMapping      next node
             │
             └──────► back to RECURSION_ENTRY
```

### 5.2 Recursion Configuration

```typescript
/**
 * @title RecursionConfig
 * @notice Controls how a workflow feeds output back as input.
 * @dev Constitutional safeguards prevent infinite loops and
 *      optimization addiction.
 */
interface RecursionConfig {
  /** Hard cap on iterations. Non-negotiable. */
  maxIterations: number;

  /**
   * Condition to terminate recursion.
   * Evaluated after each iteration.
   * Expression format: JSONPath or JS expression operating on the output items.
   * Example: "output.qualityScore >= 0.95"
   */
  terminationCondition: string;

  /**
   * How to feed output back as input.
   * Maps output fields to input fields for the next iteration.
   * Example: { "refinedResult": "inputData", "qualityScore": "priorScore" }
   */
  feedbackMapping: Record<string, string>;

  /**
   * Constitutional checkpoint interval.
   * Every N iterations, the workflow verifies it is still operating
   * within the constitutional space. This prevents optimization addiction —
   * endlessly refining toward a local maximum that has drifted from
   * the constitutional field.
   */
  convergenceCheckInterval: number;

  /**
   * What to check at the constitutional checkpoint.
   * Submitted as a convergence request if the check fails.
   */
  constitutionalCheckCriteria: string[];

  /**
   * Strategy when max iterations hit without termination.
   * 'output_best' returns the highest-quality iteration so far.
   * 'converge' submits the situation for convergence.
   */
  maxIterationStrategy: 'output_best' | 'converge';

  /**
   * Metric to track across iterations for quality comparison.
   * Used by 'output_best' strategy and for Learning Substrate reporting.
   */
  qualityMetric: string;
}
```

### 5.3 Recursion Safeguards

Recursive workflows are the most dangerous pattern in the system. Without safeguards, they can consume infinite resources optimizing toward increasingly marginal gains — a micro-instance of the Great Filter itself (endless optimization).

| Safeguard | Mechanism |
|-----------|-----------|
| Hard iteration cap | `maxIterations` — cannot be overridden at runtime |
| Resource ceiling | Workflow `resourceLimits` enforced across all iterations |
| Constitutional checkpoint | Every N iterations, verify the refinement direction is still within the constitutional field |
| Quality plateau detection | If quality metric improves less than a threshold between iterations, terminate |
| Convergence on exhaustion | If max iterations hit, the system converges rather than silently outputting a potentially insufficient result |

---

## 6. Branching Execution

Branching is how B.R.A.N.D.I. parallelizes work. A single workflow forks into multiple execution paths, each handled by independent agent clusters.

### 6.1 Branch Lifecycle

```
                    ┌───────────┐
                    │  BRANCH   │
                    │  NODE     │
                    └─────┬─────┘
                          │
              ┌───────────┼───────────┐
              │           │           │
              ▼           ▼           ▼
         ┌────────┐  ┌────────┐  ┌────────┐
         │Path A  │  │Path B  │  │Path C  │
         │        │  │        │  │        │
         │Agent   │  │Agent   │  │Agent   │
         │Cluster │  │Cluster │  │Cluster │
         │  α     │  │  β     │  │  γ     │
         │        │  │        │  │        │
         │Nodes:  │  │Nodes:  │  │Nodes:  │
         │A1→A2→A3│  │B1→B2   │  │C1→C2→C3│
         │        │  │        │  │→C4     │
         └───┬────┘  └───┬────┘  └───┬────┘
             │            │           │
             └────────────┼───────────┘
                          │
                          ▼
                 ┌─────────────────┐
                 │   CONSENSUS     │
                 │   BOUNDARY      │
                 │                 │
                 │  Aggregator     │
                 │  merges results │
                 └────────┬────────┘
                          │
                          ▼
                   Continue workflow
```

### 6.2 Branch Node Behavior

When the workflow engine reaches a BRANCH node:

```typescript
/**
 * @notice Branch execution protocol
 *
 * 1. BRANCH node receives input WorkflowItem[]
 * 2. Branch configuration determines how items are distributed:
 *    - 'duplicate': every path gets a copy of all items
 *    - 'partition': items are split across paths by condition
 *    - 'assign': each path gets specifically assigned items
 * 3. A Spawner agent creates an agent cluster for each path
 * 4. Each path begins executing independently (asynchronous)
 * 5. Paths have no visibility into each other's state
 * 6. Each path terminates at the corresponding CONSENSUS_BOUNDARY
 */

interface BranchConfig {
  /** How to distribute items to paths */
  distribution: 'duplicate' | 'partition' | 'assign';

  /** Path definitions */
  paths: BranchPath[];

  /** The consensus boundary node that collects results */
  consensusBoundaryNodeId: string;
}

interface BranchPath {
  /** Path identifier */
  pathId: string;

  /** Human-readable label */
  label: string;

  /** First node in this path */
  entryNodeId: string;

  /** Condition for item routing (for 'partition' distribution) */
  condition: string | null;

  /** Resource budget for this path (subset of workflow budget) */
  resourceBudget: Partial<ResourceLimit>;
}
```

### 6.3 Branch State Tracking

```typescript
/**
 * @title BranchState
 * @notice Tracks the execution state of a single parallel branch.
 */
interface BranchState {
  /** Branch path identifier */
  pathId: string;

  /** Current status */
  status: 'running' | 'completed' | 'failed' | 'awaiting_convergence';

  /** Agent cluster assigned to this branch */
  clusterAgentIds: string[];

  /** Items this branch received */
  inputItems: WorkflowItem[];

  /** Items this branch produced (available when completed) */
  outputItems: WorkflowItem[];

  /** Resource usage within this branch */
  resourceUsage: ResourceUsage;

  /** Timing */
  startedAt: number;
  completedAt: number | null;
}
```

---

## 7. Consensus Boundaries

A consensus boundary is where parallel branches must synchronize. It is the control point that prevents the swarm from fragmenting into incoherent parallel realities.

### 7.1 Consensus Boundary Protocol

```typescript
/**
 * @notice Consensus boundary execution protocol
 *
 * 1. Consensus boundary node activates when ALL connected
 *    branch paths have completed (or entered contentment protocol)
 * 2. An Aggregator agent receives all branch outputs
 * 3. Aggregator synthesizes results using configured strategy
 * 4. Aggregated result is evaluated:
 *    a. Do the branch results agree? → proceed
 *    b. Do they contradict? → policy determines response
 *    c. Is the aggregated result beyond scope? → convergence
 * 5. If proceeding: output merged WorkflowItem[] to next node
 * 6. If converging: submit convergence request, workflow enters
 *    AWAITING_CONVERGENCE status
 */
```

### 7.2 Consensus Boundary Configuration

```typescript
interface ConsensusBoundaryConfig {
  /** Which branch paths feed into this boundary */
  sourcePathIds: string[];

  /** Aggregation strategy */
  strategy: AggregationStrategy;

  /** How to handle incomplete branches */
  incompleteBranchPolicy: IncompleteBranchPolicy;

  /** How to handle contradictions between branches */
  contradictionPolicy: ContradictionPolicy;
}

enum AggregationStrategy {
  /** Merge all results into a single item set */
  MERGE = 'merge',

  /** Each branch votes on the result, majority wins */
  VOTE = 'vote',

  /** Results weighted by branch-specific quality metrics */
  WEIGHTED = 'weighted',

  /** Custom aggregation logic (implemented by the Aggregator agent) */
  CUSTOM = 'custom',
}

enum IncompleteBranchPolicy {
  /** Wait indefinitely for all branches (no dead states — mutualism) */
  WAIT = 'wait',

  /** Proceed with quorum (minimum N branches completed) */
  QUORUM = 'quorum',

  /** Proceed with whatever is available after timeout, flag gaps */
  BEST_EFFORT = 'best_effort',
}

enum ContradictionPolicy {
  /** Submit to convergence — let Admin/Counsel/Human resolve */
  CONVERGE = 'converge',

  /** Majority result wins (only valid for VOTE strategy) */
  MAJORITY = 'majority',

  /** Abort the workflow */
  ABORT = 'abort',

  /** Include all contradicting results in output, flagged */
  INCLUDE_ALL = 'include_all',
}
```

### 7.3 What "All Branches Complete" Means

A branch is considered complete when:

- Its final node has produced output, **OR**
- It has entered a convergence gate and received a directive, **OR**
- It has reverted to iterative mutualism (the mutualism context is treated as the branch output)

A branch is never considered "timed out" in the traditional sense. If a branch is taking long, the system investigates why — which is itself iterative mutualism. The Sentinel agent monitoring the cluster detects the delay and surfaces the first logistical reality preventing completion.

---

## 8. Data Flow

### 8.1 The Item Model

Data flows through workflows as `WorkflowItem[]` — an array of JSON objects with provenance metadata. This is consistent with n8n's core data model but extended for B.R.A.N.D.I.'s requirements.

**How items move through a workflow:**

```
    Node A outputs:     [{data: {...}, provenance: {...}},
                         {data: {...}, provenance: {...}}]
         │
         │ Connection applies:
         │   - condition filter (drop items that don't match)
         │   - data mapping (transform fields)
         │   - provenance update (add transformation record)
         │
         ▼
    Node B receives:    [{data: {...mapped...}, provenance: {...updated...}}]
```

Each node processes every item it receives individually. A node that receives 10 items runs its operation 10 times (unless it's a batch-aware node like MERGE or CONSENSUS_BOUNDARY).

### 8.2 Provenance Tracking

Every transformation is recorded. The provenance chain is append-only.

```typescript
/**
 * @notice When a node processes an item:
 *
 * 1. The node reads the input item
 * 2. The node performs its operation
 * 3. The node produces output item(s)
 * 4. The engine appends a TransformationRecord to each output item's
 *    provenance.transformationChain
 * 5. The output item's provenance.originAgentId remains unchanged
 *    (always points to the agent that created the original data)
 *
 * This creates an auditable chain from origin to current state.
 */
function appendProvenance(
  item: WorkflowItem,
  nodeId: string,
  agentId: string,
  transformationType: string,
): WorkflowItem {
  const inputHash = computeHash(item.data);

  return {
    ...item,
    provenance: {
      ...item.provenance,
      transformationChain: [
        ...item.provenance.transformationChain,
        {
          nodeId,
          agentId,
          transformationType,
          timestamp: Date.now(),
          inputHash,
        },
      ],
    },
  };
}
```

### 8.3 Chain Commitment

Items flagged with `requiresChainCommitment: true` have their hashes submitted to N.I.K.O.System's OracleFacet. This provides tamper-proof proof that the data existed in this state at this time.

```typescript
/**
 * @notice Chain commitment flow
 *
 * 1. Node produces output item with requiresChainCommitment: true
 * 2. Engine computes hash of (item.data + item.provenance)
 * 3. Engine calls niko.submitResult(chainId, resultHash, proof)
 * 4. OracleFacet records the hash on-chain
 * 5. Transaction hash is attached to the item's metadata
 *
 * The full item content stays off-chain (in PostgreSQL).
 * The on-chain hash proves the content existed and hasn't been altered.
 */
```

### 8.4 Item Branching and Merging

When items hit a BRANCH node:

```
Input: [{item1}, {item2}, {item3}]

Distribution: 'duplicate'
  Path A gets: [{item1}, {item2}, {item3}]    (copies)
  Path B gets: [{item1}, {item2}, {item3}]    (copies)

Distribution: 'partition' (condition: item.data.region)
  Path A (region=US): [{item1}]
  Path B (region=EU): [{item2}, {item3}]

Distribution: 'assign' (explicit mapping)
  Path A: [{item1}, {item3}]
  Path B: [{item2}]
```

When items reach a CONSENSUS_BOUNDARY (MERGE):

```
Path A output: [{resultA1}, {resultA2}]
Path B output: [{resultB1}]

Strategy: 'merge'
  Output: [{resultA1}, {resultA2}, {resultB1}]
  (all items combined, provenance preserved showing which path produced each)

Strategy: 'weighted'
  Output: [{weightedResult}]
  (Aggregator agent produces a single synthesized item)
```

---

## 9. Error Handling

### 9.1 Node-Level Errors

When a node fails:

1. The node's `NodeExecutionState.status` is set to `'failed'`.
2. The error is logged with full context (input items, agent state, error details).
3. The workflow engine evaluates the error against the node's retry policy.
4. If retries remain: re-execute the node.
5. If retries exhausted: evaluate the workflow's error policy.

```typescript
interface NodeErrorPolicy {
  /** Maximum retry attempts */
  maxRetries: number;

  /** Backoff between retries */
  retryBackoff: 'none' | 'linear' | 'exponential';

  /** Base delay for backoff (ms) */
  baseDelayMs: number;

  /** What to do if all retries fail */
  onExhaustion: 'fail_workflow' | 'skip_node' | 'converge';
}
```

### 9.2 Workflow-Level Errors

When a workflow fails (node exhausted retries with `fail_workflow` policy):

1. Workflow status set to `FAILED`.
2. All running agents in the workflow are notified to wind down.
3. Full execution state is persisted (for debugging and Learning Substrate).
4. A report is submitted to S.H.A.N.N.O.N. via NIKO.
5. If the workflow is part of a larger convergence process, the failure is surfaced as a logistical reality preventing convergence → iterative mutualism kicks in.

### 9.3 There Are No Silent Failures

Every failure produces:

- A log entry with full context
- An update to the workflow execution record
- A report to the Learning Substrate
- An event on the RxJS Event Bus (for Sentinels to detect patterns)

Failures are not hidden. They are training data.

---

## 10. Workflow Composition

Workflows can reference other workflows, enabling modular composition.

### 10.1 Sub-Workflows

A node can reference another workflow definition as its operation. When the engine reaches this node, it starts a new `WorkflowExecution` for the sub-workflow and passes the current items as trigger data.

```typescript
interface SubWorkflowNodeConfig {
  /** The workflow ID to execute as a sub-workflow */
  subWorkflowId: string;

  /** How to handle sub-workflow output */
  outputMapping: 'pass_through' | 'merge' | 'custom';

  /** Resource budget allocated to the sub-workflow (from parent's remaining budget) */
  resourceBudget: Partial<ResourceLimit>;
}
```

### 10.2 Composition Constraints

- Sub-workflow resource budgets come from the parent's remaining budget (no resource creation).
- Sub-workflow recursion depth is counted globally (a recursive sub-workflow inside a recursive parent shares the max iteration cap).
- Sub-workflows inherit the parent's constitutional hash (they operate in the same constitutional field state).
- Circular sub-workflow references are detected at definition time and rejected.

---

## 11. Implementation Guidance

### 11.1 Build Order

```
1. WorkflowItem type + provenance types
   └── 2. Node types + connection types
       └── 3. Trigger types + normalization
           └── 4. WorkflowDefinition + WorkflowExecution types
               └── 5. Basic execution engine (linear workflows only)
                   ├── 6a. Condition nodes (branching logic)
                   ├── 6b. Chain commit integration
                   └── 6c. Error handling + retry
                       └── 7. BRANCH + CONSENSUS_BOUNDARY
                           └── 8. Recursion (RECURSION_ENTRY + EXIT)
                               └── 9. CONVERGENCE_GATE integration
                                   └── 10. Sub-workflow composition
```

Start with linear workflows. Get items flowing through nodes with provenance. Then add branching. Then recursion. Then convergence gates. Each layer builds on the last.

### 11.2 Testing Workflows

| Test Category | What to Verify |
|--------------|----------------|
| Linear execution | Items flow through nodes in order, provenance accumulates |
| Condition routing | Items route to correct branch based on condition evaluation |
| Trigger normalization | Every trigger type produces valid WorkflowItem[] |
| Branch distribution | duplicate/partition/assign all distribute correctly |
| Consensus boundary | Aggregator waits for all branches, merges correctly |
| Contradiction detection | Aggregator detects conflicting branch results |
| Recursion termination | Every termination condition works, max iterations enforced |
| Constitutional checkpoint | Recursion pauses for checkpoint at configured interval |
| Convergence gate | Workflow pauses, resumes on directive, handles mutualism reversion |
| Chain commitment | Hashes match, on-chain records created |
| Provenance integrity | Full transformation chain is accurate and append-only |
| Error retry | Retries fire with correct backoff, exhaustion policy enforced |
| Resource accounting | Usage tracked accurately, ceilings enforced |
| Sub-workflow | Budget inheritance, recursion depth sharing, circular reference detection |

---

**Copyright © 2024 John A. Welch, Director, Blinded Eye Foundation. All rights reserved.**
