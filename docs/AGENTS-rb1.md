# B.R.A.N.D.I. — Agent Specification

**Copyright © 2024 John A. Welch, Director, Blinded Eye Foundation. All rights reserved.**

**Prerequisites:** Read `DESIGN.md` then `docs/ARCHITECTURE.md` before this document.

---

## 1. The Agent Primitive

A B.R.A.N.D.I. agent is the atomic unit of the swarm. Every agent — regardless of type — shares the same fundamental architecture. Specialization happens through capability declaration and tool access, not through structural differences.

### 1.1 The Single-Loop Operating Model

A B.R.A.N.D.I. agent does not have modes. It does not switch between "normal operation" and "escalation" and "waiting." It has **one loop**. The operating principle is constant: operate within the constitutional field through iterative mutualism. What changes is the scale of engagement, not the nature of it.

```
                    ┌──────────────────────┐
                    │     RECEIVE INPUT    │
                    │  (WorkflowItem[] or  │
                    │   Directive or       │
                    │   Trigger Event)     │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │   ASSESS ACTION      │
                    │                      │
                    │ What does this input  │
                    │ require me to do?    │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │  BOUNDARY CHECK      │
                    │                      │
                    │ Is the required      │
                    │ action within my     │
                    │ declared scope?      │
                    └────┬────────────┬────┘
                         │            │
                    WITHIN       EXCEEDS
                    SCOPE        SCOPE
                         │            │
              ┌──────────▼──┐   ┌─────▼──────────┐
              │   EXECUTE   │   │ REPORT & WAIT  │
              │             │   │                │
              │ Use tools.  │   │ Package context│
              │ Transform   │   │ Submit to NIKO │
              │ data.       │   │ Await directive│
              │ Do the work.│   │ from convergence│
              └──────┬──────┘   └────────┬───────┘
                     │                   │
                     │            ┌──────▼───────┐
                     │            │  DIRECTIVE    │
                     │            │  RECEIVED     │
                     │            │               │
                     │            │ Or: mutualism │
                     │            │ reversion     │
                     │            │ (new input)   │
                     │            └──────┬────────┘
                     │                   │
                     └─────────┬─────────┘
                               │
                    ┌──────────▼───────────┐
                    │      REPORT          │
                    │                      │
                    │ Submit AgentReport   │
                    │ to NIKO → SHANNON    │
                    │ Learning Substrate   │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │   NEXT INPUT         │
                    │                      │
                    │ Return to top of     │
                    │ loop. Process next   │
                    │ WorkflowItem or      │
                    │ await next trigger.  │
                    └──────────────────────┘
```

**Key insight:** When the agent hits a boundary and enters "report & wait," it has not left the loop. The directive or mutualism reversion that comes back IS the next input. The loop never breaks. The loop never forks into a separate mode. The agent is always doing the same thing at different scales.

### 1.2 What Every Agent Has

Regardless of type, every agent possesses:

| Component | Purpose |
|-----------|---------|
| **Identity** | `AgentIdentity` — on-chain registration, type, name, capabilities, scope |
| **Runtime** | `IAgentRuntime` — the execution environment implementing the single loop |
| **Knowledge Interface** | Connection to S.H.A.N.N.O.N. via NIKO for curated data |
| **Execution Interface** | Tools, API connections, workflow actions — scoped by declaration |
| **Boundary Detector** | Evaluates proposed actions against declared scope |
| **Convergence Channel** | Submit convergence requests to NIKO, receive directives back |
| **Report Channel** | Submit outcomes to NIKO → S.H.A.N.N.O.N. Learning Substrate |

### 1.3 What No Agent Has

No agent possesses:

- Direct access to S.H.A.N.N.O.N. (everything routes through NIKO)
- Constitutional evaluation capability (the field handles that, not the agent)
- Authority over other agents (the swarm has no hierarchy)
- Ability to override its own scope (scope is set at registration)
- Access to another agent's working memory (agents are isolated)
- Ability to force convergence or bypass it

---

## 2. Agent Types

Each agent type is a specialization of the primitive — same loop, same interfaces, different capabilities and tools.

### 2.1 Executor Agent

**Purpose:** Performs defined tasks within a workflow. The workhorse.

**Analogous to:** n8n's Tools Agent.

**Capabilities:**
- Interacts with external APIs and services
- Selects appropriate tools from its declared tool set based on the task
- Transforms data between formats
- Performs CRUD operations on external systems

**Scope characteristics:**
- Typically narrow and well-defined
- `requiresConvergence` is false for routine operations
- `hasSideEffects` varies by tool
- Cannot spawn sub-agents (`canSpawn: false`)
- Usually cannot interact with humans directly (`canInteractHuman: false`)

**When boundary is hit:** Packages the task context, the tool it intended to use, and the reason it can't proceed. Reports to NIKO and waits.

```typescript
/**
 * @title ExecutorAgent
 * @notice Specialized agent for task execution.
 * @dev The most common agent type. Handles routine workflow
 *      operations: API calls, data transforms, CRUD.
 */
interface ExecutorAgentConfig {
  /** The tools this executor has access to */
  tools: ToolDeclaration[];

  /** Maximum retry attempts for failed tool operations */
  maxRetries: number;

  /** Backoff strategy for retries */
  retryBackoff: 'linear' | 'exponential';

  /** Whether to report every action or only significant ones */
  reportingLevel: 'all' | 'significant' | 'errors_only';
}
```

### 2.2 Conversational Agent

**Purpose:** Facilitates human-facing interactions. The human's window into the swarm.

**Analogous to:** n8n's Conversational Agent.

**Capabilities:**
- Maintains conversation context
- Understands human intent through natural language
- Translates between human language and agent operations
- Presents convergence requests to the human participant in understandable form
- Receives human evaluations for the convergence process

**Scope characteristics:**
- `canInteractHuman: true` (the defining trait)
- Broad domain awareness but limited direct execution ability
- Often delegates to Executor agents for actual task completion
- `requiresConvergence` typically false for conversation, true for committing to actions on the human's behalf

**Critical design note:** The Conversational Agent does NOT position itself as an authority or advisor. It is a communication interface. It presents information, relays convergence requests, and receives human input. It does not filter, interpret, or editorialize the human's evaluation. The human's voice passes through unaltered.

```typescript
/**
 * @title ConversationalAgentConfig
 * @notice Configuration for human-facing agent.
 * @dev This agent is the primary interface between the swarm
 *      and the human participant. It must treat the human
 *      as an equal participant — never as a supervisor,
 *      never as a subordinate.
 */
interface ConversationalAgentConfig {
  /** Conversation history retention limit */
  maxHistoryItems: number;

  /** Supported interaction channels */
  channels: ('cli' | 'web' | 'api' | 'chat')[];

  /** Whether to present raw convergence data or summarized */
  convergencePresentation: 'raw' | 'summarized' | 'both';

  /** Language/locale preferences */
  locale: string;
}
```

### 2.3 ReAct Agent

**Purpose:** Breaks down complex problems into structured reasoning-and-action sequences.

**Analogous to:** n8n's ReAct Agent (Reason and Act model).

**Capabilities:**
- Decomposes complex tasks into step sequences
- Reasons about intermediate results before proceeding
- Adjusts its plan based on observations
- Requests knowledge from S.H.A.N.N.O.N. between steps to fill gaps

**Scope characteristics:**
- Broader scope than Executors — handles multi-step operations
- Higher resource ceilings to accommodate reasoning chains
- `requiresConvergence` triggered by accumulated impact, not single actions
- May request convergence mid-sequence if reasoning reveals unexpected scope

**The ReAct loop within the single loop:**

The ReAct agent's specialization is that its EXECUTE phase contains its own internal cycle: Think → Act → Observe → Think → Act → Observe... This does not break the single-loop model. The ReAct cycle IS the execution. The boundary check still happens before each Act. The report still happens at the end.

```typescript
/**
 * @title ReActAgentConfig
 * @notice Configuration for reason-and-act agent.
 */
interface ReActAgentConfig {
  /** Maximum reasoning steps before requiring convergence */
  maxReasoningSteps: number;

  /** Knowledge query budget per task */
  maxKnowledgeQueries: number;

  /** Whether to log intermediate reasoning (for Learning Substrate) */
  logIntermediateReasoning: boolean;

  /** Accumulated impact threshold that triggers convergence */
  impactThreshold: ImpactLevel;
}
```

### 2.4 Spawner Agent

**Purpose:** Dynamically creates sub-agents to handle parallel workloads. Manages the "Branching" in B.R.A.N.D.I.

**Capabilities:**
- Analyzes a task and determines it requires parallel execution
- Defines sub-agent specifications (type, scope, tools, resource limits)
- Registers sub-agents via NIKO (OrchestratorFacet)
- Monitors sub-agent lifecycle
- Does NOT aggregate results (that's the Aggregator's job)

**Scope characteristics:**
- `canSpawn: true` (the defining trait)
- `maxSpawnDepth` limits recursive spawning (spawned agents spawning agents)
- `maxSubAgents` limits total spawn count
- Spawning itself may require convergence if the spawn count or resource allocation is high

**Critical constraint:** A Spawner agent cannot grant a sub-agent more scope than it possesses. Scope only narrows downward. A sub-agent's `authorizedDomains` must be a subset of the parent's. A sub-agent's `resourceCeiling` must be within the parent's remaining budget.

```typescript
/**
 * @title SpawnerAgentConfig
 * @notice Configuration for agent that dynamically creates sub-agents.
 * @dev Sub-agent scope MUST be a subset of the Spawner's scope.
 *      This is enforced at registration time by the NIKO integration layer.
 */
interface SpawnerAgentConfig {
  /** Maximum concurrent sub-agents */
  maxConcurrentSubAgents: number;

  /** Maximum depth of spawning chains */
  maxSpawnDepth: number;

  /** Default resource allocation per sub-agent (as fraction of own budget) */
  defaultSubAgentBudgetFraction: number;

  /** Sub-agent types this Spawner is allowed to create */
  allowedSubAgentTypes: AgentType[];

  /** Whether to auto-deactivate sub-agents when parent task completes */
  autoDeactivateOnComplete: boolean;
}
```

### 2.5 Aggregator Agent

**Purpose:** Synthesizes results from multiple agents operating in parallel. Manages consensus boundaries in workflows.

**Capabilities:**
- Receives results from multiple agents or workflow branches
- Synthesizes, compares, and merges parallel outputs
- Detects contradictions or conflicts between branch results
- Determines whether aggregated results pass constitutional evaluation
- Triggers convergence if aggregated results reveal decisions beyond scope

**Scope characteristics:**
- Read access to results from agents in its assigned cluster
- Does not execute external actions — it processes internal results
- High `requiresConvergence` threshold — aggregation that reveals novel decisions should converge

**Distinction from convergence:** Aggregation is operational — combining parallel computation results. Convergence is decisional — manifesting a decision through Admin/Counsel/Human alignment. An Aggregator may trigger convergence but does not perform it.

```typescript
/**
 * @title AggregatorAgentConfig
 * @notice Configuration for result synthesis agent.
 */
interface AggregatorAgentConfig {
  /** Agent IDs this aggregator collects from */
  sourceAgentIds: string[];

  /** Aggregation strategy */
  strategy: 'merge' | 'vote' | 'weighted' | 'custom';

  /** How to handle contradictions between sources */
  contradictionPolicy: 'flag_convergence' | 'majority' | 'abort';

  /** Timeout for waiting on all sources (0 = no timeout, wait indefinitely) */
  sourceTimeoutMs: number;

  /** Minimum number of sources required before aggregation proceeds */
  quorum: number;
}
```

### 2.6 Sentinel Agent

**Purpose:** Monitors swarm health and detects systemic issues no individual agent would recognize. The swarm's immune system.

**Capabilities:**
- Monitors agent lifecycle events across the swarm
- Detects patterns: cascading failures, resource exhaustion, scope drift
- Reports anomalies to MonitoringFacet on-chain
- Triggers convergence for systemic threats
- Does NOT intervene directly — it observes and reports

**Scope characteristics:**
- Broad observational scope across the swarm
- No execution tools — observation and reporting only
- Always-on, long-running
- `canInteractHuman: false` (reports through the system, not to the human directly)

**What Sentinels watch for:**

| Pattern | Description | Response |
|---------|-------------|----------|
| Cascade failure | Multiple agents entering ERROR in sequence | Log event, trigger convergence |
| Resource exhaustion | Cluster approaching resource ceiling | Log event, alert |
| Scope drift | Agent actions trending outside declared domains | Log event, flag for review |
| Convergence bottleneck | Convergence requests accumulating without resolution | Log event, surface first logistical reality |
| Spawn storms | Spawner creating sub-agents faster than they complete | Log event, throttle recommendation |
| Knowledge gaps | Multiple agents querying the same missing knowledge | Log event, feed to Learning Substrate |

```typescript
/**
 * @title SentinelAgentConfig
 * @notice Configuration for swarm health monitoring agent.
 */
interface SentinelAgentConfig {
  /** Which agent clusters to monitor */
  monitoredClusters: string[];

  /** Polling interval for health checks (ms) */
  healthCheckIntervalMs: number;

  /** Anomaly detection sensitivity (0-1, higher = more sensitive) */
  anomalySensitivity: number;

  /** Event types to watch for */
  watchedEventTypes: BrandiEventType[];

  /** Minimum severity to report */
  minReportSeverity: 'info' | 'warning' | 'error' | 'critical';
}
```

---

## 3. Agent Lifecycle

### 3.1 Registration

Every agent begins with registration through N.I.K.O.System's OrchestratorFacet.

```typescript
/**
 * @notice Registration flow
 *
 * 1. Agent specification is defined (type, capabilities, scope, tools)
 * 2. Metadata is uploaded to IPFS (or equivalent content-addressable store)
 * 3. OrchestratorFacet.registerAgent(agentId, agentAddress, metadataUri) is called
 * 4. Agent enters IDLE status
 * 5. AgentRegistered event emitted on-chain
 * 6. BRANDI_AGENT_REGISTERED event published to RxJS Event Bus
 */
```

**Pre-registration validation:**

Before calling `registerAgent`, the NIKO integration layer validates:

- `agentId` is unique (not already registered)
- `agentAddress` is not the zero address
- Capabilities and scope are internally consistent
- If the agent has a parent (spawned), its scope is a subset of the parent's
- Resource limits are within the parent's remaining budget (if spawned)
- Tool declarations reference valid, available tools

### 3.2 Status Transitions

```
                 register
                    │
                    ▼
    ┌──────────► IDLE ◄──────────────────┐
    │               │                    │
    │          start task                │
    │               │                recovery
    │               ▼                    │
    │           RUNNING ────────────► ERROR
    │            │    │                  │
    │       pause│    │boundary hit      │
    │            │    │(report & wait)   │
    │            ▼    │                  │
    │         PAUSED  │                  │
    │            │    │                  │
    │       resume    │                  │
    │            │    │                  │
    │            ▼    │                  │
    │         RUNNING◄┘                  │
    │                                    │
    │         deactivate                 │
    │            │                       │
    │            ▼                       │
    └─────── INACTIVE                   │
              (preserved                │
               history)                 │
                                        │
    Note: ERROR → IDLE requires         │
    investigation + resolution ─────────┘
```

**Status meanings:**

| Status | Meaning | Agent behavior |
|--------|---------|---------------|
| IDLE | Registered, no active task | Awaiting input. The loop is at "RECEIVE INPUT." |
| RUNNING | Actively executing | In the ASSESS → BOUNDARY CHECK → EXECUTE portion of the loop. |
| PAUSED | Temporarily suspended | External pause (system maintenance, resource rebalancing). Not the same as report-and-wait. |
| ERROR | Something failed | Requires investigation. Agent does not self-recover from ERROR to RUNNING. ERROR → IDLE requires explicit resolution. |
| INACTIVE | Deactivated | Removed from active swarm. On-chain history preserved. Cannot be reactivated (register a new agent instead). |

**Important distinction:** "Report and wait" (boundary hit) is NOT a status transition. The agent remains in RUNNING status while awaiting a convergence directive. The agent is still in its loop — it's at the "await directive" step. The loop is not broken; the agent is not stuck. It is contentedly operating within iterative mutualism.

### 3.3 Deactivation

Agents are deactivated, never deleted. Deactivation:

1. Calls `OrchestratorFacet.deactivateAgent(agentId)`
2. Agent status becomes INACTIVE
3. On-chain history (actions, metrics) is preserved permanently
4. Off-chain state (working memory, context) is archived then cleared
5. If the agent has sub-agents, they are also deactivated (cascade)
6. The Learning Substrate retains all reports from this agent

---

## 4. Boundary Detection

The boundary detector is the mechanism that keeps agents honest about their limits. It evaluates every proposed action against the agent's declared scope before execution.

### 4.1 Detection Algorithm

```typescript
/**
 * @title detectBoundary
 * @notice Evaluates whether a proposed action is within the agent's scope.
 * @dev Called before every action in the single loop.
 *      This is NOT constitutional validation — that's S.H.A.N.N.O.N.'s job.
 *      This is scope enforcement — "am I allowed to do this?"
 *
 * @param action The action the agent proposes to take
 * @returns BoundaryCheckResult indicating proceed or request_convergence
 */
function detectBoundary(
  action: ProposedAction,
  scope: AgentScope,
  currentResourceUsage: ResourceUsage,
): BoundaryCheckResult {

  // Check 1: Domain authorization
  // Are any of the affected domains outside my authorized domains?
  const unauthorizedDomains = action.affectedDomains.filter(
    d => !scope.authorizedDomains.includes(d)
  );
  if (unauthorizedDomains.length > 0) {
    return {
      withinScope: false,
      reason: `Action affects unauthorized domains: ${unauthorizedDomains.join(', ')}`,
      recommendation: 'request_convergence',
    };
  }

  // Check 2: Resource limits
  // Would this action exceed my resource ceiling?
  if (action.estimatedCost.maxGasPerAction &&
      action.estimatedCost.maxGasPerAction > scope.resourceCeiling.maxGasPerAction) {
    return {
      withinScope: false,
      reason: `Estimated gas ${action.estimatedCost.maxGasPerAction} exceeds ceiling ${scope.resourceCeiling.maxGasPerAction}`,
      recommendation: 'request_convergence',
    };
  }

  // Check 3: Cumulative resource usage
  // Would this push my total resource usage beyond workflow limits?
  const projectedTotal = currentResourceUsage.totalGas +
    (action.estimatedCost.maxGasPerAction || 0n);
  if (projectedTotal > scope.resourceCeiling.maxGasPerWorkflow) {
    return {
      withinScope: false,
      reason: `Cumulative resource usage would exceed workflow ceiling`,
      recommendation: 'request_convergence',
    };
  }

  // Check 4: Explicit convergence requirements
  // Is this action type in the always-converge list?
  if (scope.alwaysConverge.some(pattern => action.description.match(pattern))) {
    return {
      withinScope: false,
      reason: `Action matches always-converge pattern`,
      recommendation: 'request_convergence',
    };
  }

  // Check 5: Tool-level convergence
  // Do any of the tools being used require convergence?
  const convergenceTools = action.toolIds.filter(
    id => scope.tools?.find(t => t.toolId === id)?.requiresConvergence
  );
  if (convergenceTools.length > 0) {
    return {
      withinScope: false,
      reason: `Tools require convergence: ${convergenceTools.join(', ')}`,
      recommendation: 'request_convergence',
    };
  }

  // Check 6: Side effect awareness
  // Side effects in unfamiliar territory should converge
  if (action.hasSideEffects && unauthorizedDomains.length > 0) {
    return {
      withinScope: false,
      reason: `Side effects in unauthorized domains`,
      recommendation: 'request_convergence',
    };
  }

  // All checks passed
  return {
    withinScope: true,
    reason: null,
    recommendation: 'proceed',
  };
}
```

### 4.2 What Happens at a Boundary

When `detectBoundary` returns `recommendation: 'request_convergence'`:

1. The agent packages a `ConvergenceRequest` with full decision context.
2. The agent submits the request to NIKO via `niko.submitConvergenceRequest()`.
3. The agent calls `niko.awaitDirective(requestId)` which blocks on a WebSocket.
4. The agent remains in RUNNING status. It is not paused. It is in the loop.
5. When a `ConvergenceDirective` arrives, the agent processes it as new input.
6. If iterative mutualism reverts, the agent receives a `MutualismContext` with a new task — addressing the first logistical reality.

The agent does not decide what to do about its boundary. It does not improvise. It does not guess. It reports and waits. The convergence process happens above it.

---

## 5. Agent Spawning

Spawner agents create sub-agents at runtime. This is how B.R.A.N.D.I. scales dynamically.

### 5.1 Spawn Protocol

```
Spawner Agent                    NIKO Integration Layer
     │                                    │
     │── Define sub-agent spec ──────────►│
     │   (type, scope, tools, budget)     │
     │                                    │── Validate scope subset
     │                                    │── Validate resource budget
     │                                    │── Validate spawn depth
     │                                    │
     │                              VALID │ INVALID
     │                                    │
     │                                    │── Register via OrchestratorFacet
     │                                    │── Publish AGENT_REGISTERED event
     │                                    │
     │◄── SubAgentHandle ────────────────│
     │   (agentId, status subscription)   │
     │                                    │
     │── Assign task to sub-agent ───────►│
     │                                    │── Route task to sub-agent
     │                                    │
```

### 5.2 Spawn Constraints

| Constraint | Rule | Enforcement |
|-----------|------|-------------|
| Scope inheritance | Sub-agent scope ⊆ parent scope | NIKO integration layer validates at registration |
| Resource budget | Sub-agent budget ≤ parent's remaining budget | NIKO integration layer validates at registration |
| Spawn depth | Each spawn increments depth counter. Cannot exceed `maxSpawnDepth` | NIKO integration layer validates at registration |
| Type restriction | Spawner can only create types in its `allowedSubAgentTypes` | NIKO integration layer validates at registration |
| Cascade deactivation | When parent deactivates, all sub-agents deactivate | OrchestratorFacet enforces |

### 5.3 Spawn Depth

Spawn depth prevents infinite agent recursion:

```
Root Agent (depth 0)
  └── Spawned Agent A (depth 1)
       └── Spawned Agent A1 (depth 2)
            └── Spawned Agent A1a (depth 3)
                 └── BLOCKED if maxSpawnDepth = 3
```

Each spawned agent carries `parentAgentId` linking to its creator. This forms a tree that can be walked for auditing and cascade operations.

---

## 6. Agent Clusters

Agents don't just operate individually — they form clusters for coordinated parallel work.

### 6.1 Cluster Composition

A cluster is a logical grouping of agents working on related tasks. A typical cluster:

```
┌─────────────────────────────────────────┐
│           Agent Cluster α               │
│                                         │
│  ┌──────────┐    ┌──────────────────┐   │
│  │ Spawner  │───►│ Executor (×N)    │   │
│  └──────────┘    └────────┬─────────┘   │
│                           │             │
│                  ┌────────▼─────────┐   │
│                  │   Aggregator     │   │
│                  └────────┬─────────┘   │
│                           │             │
│  ┌──────────┐             │             │
│  │ Sentinel │─── monitors all ──────    │
│  └──────────┘                           │
│                                         │
│  ┌──────────────┐                       │
│  │ Conversational│ (if human            │
│  │ Agent         │  interaction needed) │
│  └──────────────┘                       │
└─────────────────────────────────────────┘
```

The Spawner forks Executors for parallel tasks. The Aggregator collects their results. The Sentinel monitors the cluster's health. A Conversational Agent is attached if the cluster's work involves human participation.

### 6.2 Cluster Formation

Clusters form dynamically based on workflow requirements. When a workflow's branching node activates:

1. The workflow engine identifies the parallel tasks.
2. A Spawner agent is assigned (or spawned) to create the cluster.
3. The Spawner registers Executor agents for each parallel branch.
4. An Aggregator agent is assigned to the consensus boundary.
5. A Sentinel agent begins monitoring the cluster.

Clusters dissolve when their consensus boundary completes and the Aggregator produces its merged result.

---

## 7. Agent-to-Agent Communication

Agents do not communicate directly with each other. All inter-agent communication is mediated by the workflow engine and N.I.K.O.System infrastructure.

### 7.1 Communication Paths

| From | To | Mechanism |
|------|-----|-----------|
| Any agent | S.H.A.N.N.O.N. | Via NIKO routing (knowledge query or report) |
| Spawner | Sub-agents | Via workflow engine task assignment |
| Executor | Aggregator | Via workflow engine result delivery at consensus boundary |
| Sentinel | Monitoring | Via NIKO MonitoringFacet |
| Conversational | Human | Via configured interaction channel (CLI, web, API) |
| Any agent | Any agent | NOT ALLOWED directly. Use workflow connections. |

### 7.2 Why No Direct Agent Communication

Direct agent-to-agent channels would:

- Create hidden dependencies the workflow engine can't track
- Make provenance tracking incomplete
- Enable emergent hierarchies (agents coordinating to bypass scope)
- Break the single-loop model (agents would need to handle unsolicited messages)
- Undermine the constitutional field (unmediated communication isn't curated)

Every piece of data that moves between agents flows through a workflow connection, which carries provenance metadata and is subject to the data flow model defined in ARCHITECTURE.md.

---

## 8. Implementation Guidance

### 8.1 Building the Agent Primitive

The implementation order for the agent system:

```
1. AgentIdentity type + AgentScope type
   └── 2. BoundaryDetector (the detection algorithm)
       └── 3. IAgentRuntime interface
           └── 4. Base AgentRuntime class (single-loop implementation)
               └── 5. NIKO client integration (knowledge, convergence, report)
                   ├── 6a. ExecutorAgent (simplest specialization)
                   ├── 6b. ConversationalAgent
                   ├── 6c. ReActAgent
                   ├── 6d. SpawnerAgent
                   ├── 6e. AggregatorAgent
                   └── 6f. SentinelAgent
```

Do not build agent types (step 6) before the base runtime (step 4) is solid. The single-loop model must work before specializations are added.

### 8.2 Testing Agents

| Test Category | What to Verify |
|--------------|----------------|
| Boundary detection | Every check in the detection algorithm triggers correctly |
| Scope enforcement | Sub-agent scope is always ⊆ parent scope |
| Single-loop integrity | Agent never exits the loop, even under error conditions |
| Report-and-wait | Agent correctly packages context and blocks on WebSocket |
| Directive processing | Agent correctly processes convergence directives as new input |
| Mutualism reversion | Agent correctly handles mutualism context as new input |
| Resource accounting | Agent tracks cumulative resource usage accurately |
| Cascade deactivation | Parent deactivation cascades to all sub-agents |
| Tool isolation | Agent cannot access tools outside its declaration |
| Memory isolation | Agent cannot read another agent's state |

### 8.3 File Headers

Every source file in the agent system must include:

```typescript
/**
 * @file [filename]
 * @title [component name]
 * @notice [what this file does]
 * @dev [implementation notes, relationship to tri-system]
 *
 * @copyright 2024 John A. Welch, Director, Blinded Eye Foundation.
 * All rights reserved.
 * @license AGPL-3.0
 *
 * Part of B.R.A.N.D.I. — Branching Recursive Asynchronous Nodal
 * Decentralized Intelligence. Operates within S.H.A.N.N.O.N.'s
 * constitutional field on N.I.K.O.System infrastructure.
 */
```

---

**Copyright © 2024 John A. Welch, Director, Blinded Eye Foundation. All rights reserved.**
