# B.R.A.N.D.I. — N.I.K.O.System Integration Specification

**Copyright © 2024 John A. Welch, Director, Blinded Eye Foundation. All rights reserved.**

**Prerequisites:** Read `DESIGN.md`, `docs/ARCHITECTURE.md`, `docs/AGENTS.md`, and `docs/WORKFLOWS.md` before this document.

---

## 1. Integration Philosophy

B.R.A.N.D.I. touches N.I.K.O.System through exactly one surface: the NIKO Integration Layer. This is not a convenience abstraction — it is an architectural constraint. No agent, no workflow node, no convergence module may access N.I.K.O.System infrastructure directly. Every request flows through `INikoClient`. Every response returns through `INikoClient`.

This constraint exists for three reasons:

1. **Provenance integrity.** If agents could bypass the integration layer, action recording and constitutional hash verification would become optional rather than structural. The integration layer makes provenance non-negotiable by mediating every interaction.

2. **Security boundary enforcement.** Individual agents do not hold on-chain roles. The B.R.A.N.D.I. system process holds `AGENT_MANAGER_ROLE`, `KERNEL_OPERATOR_ROLE`, `ORACLE_OPERATOR_ROLE`, and `MONITORING_ROLE`. The integration layer acts on behalf of agents after validating scope. This prevents any single agent from accumulating permissions through direct infrastructure access.

3. **Routing discipline.** All communication between B.R.A.N.D.I. and S.H.A.N.N.O.N. routes through N.I.K.O.System. The integration layer enforces this path. There are no shortcuts.

```
B.R.A.N.D.I. Components          NIKO Integration Layer          N.I.K.O.System
                                                                  Infrastructure
┌──────────────┐                 ┌──────────────────┐
│ Agent Runtime │────────────────►│                  │────────► OrchestratorFacet
├──────────────┤                 │                  │────────► KernelFacet
│ Workflow      │────────────────►│   INikoClient    │────────► OracleFacet
│ Engine        │                 │                  │────────► MonitoringFacet
├──────────────┤                 │   (single gate)  │────────► TreasuryFacet
│ Convergence   │────────────────►│                  │────────► NestJS Backend
│ Module        │                 │                  │────────► → S.H.A.N.N.O.N.
└──────────────┘                 └──────────────────┘
```

### 1.1 Abstraction Layers

The same operations appear at different abstraction levels across the document set. For clarity:

| Layer | Example: Agent Registration | Defined In |
|-------|---------------------------|------------|
| On-chain (Solidity) | `registerAgent(agentId, agentAddress, metadataUri)` | DESIGN.md §8.1 |
| Facet client (this doc) | `IOrchestratorFacetClient.registerAgent(agentId, agentAddress, metadataUri)` → `TransactionReceipt` | INTEGRATION.md §2.1 |
| Composite facade | `INikoClient.registerAgent(identity: AgentIdentity)` → `Promise<void>` | ARCHITECTURE.md §4.1 |

The facet client (this document) wraps the on-chain call with validation and error handling. The composite facade (`INikoClient` in ARCHITECTURE.md) wraps the facet client with event publication and cross-facet coordination. These are not competing definitions — they are layers.

### 1.2 Facets Not Specified Here

N.I.K.O.System exposes facets that B.R.A.N.D.I. does not consume directly:

| Facet | Why Excluded |
|-------|-------------|
| **ConsensusFacet** | Blockchain consensus operations. B.R.A.N.D.I. consumes consensus results via OracleFacet and chain events, but does not interact with the consensus mechanism itself. |
| **GovernanceFacet** | Multi-sig upgrade controls. B.R.A.N.D.I. does not initiate governance actions. It *reacts* to governance events (e.g., `niko.governance.upgrade_proposed` in §8.2) by triggering convergence — the system must evaluate proposed upgrades through Admin/Counsel/Human alignment before accepting them. |

These facets are infrastructure-internal. B.R.A.N.D.I.'s relationship to them is reactive (via event subscriptions), not active (via direct calls).

---

## 2. OrchestratorFacet Client

The OrchestratorFacet is B.R.A.N.D.I.'s primary on-chain interface. It manages the lifecycle of every agent in the swarm and provides tamper-proof provenance for every significant action.

### 2.1 Contract Interface (Consumed)

These are the OrchestratorFacet functions that B.R.A.N.D.I. calls. The integration layer wraps each one with scope validation, error handling, and event publication.

```typescript
/**
 * @title IOrchestratorFacetClient
 * @notice B.R.A.N.D.I.'s client interface to OrchestratorFacet.
 * @dev All calls require AGENT_MANAGER_ROLE.
 *      The integration layer holds this role — agents do not.
 */
interface IOrchestratorFacetClient {
  /**
   * @notice Register a new agent on-chain.
   * @dev Called during agent initialization. The integration layer
   *      validates the AgentIdentity before submitting.
   *      Emits AgentRegistered event on-chain.
   *      Publishes AGENT_REGISTERED to RxJS Event Bus.
   *
   * @param agentId Unique agent identifier (bytes32)
   * @param agentAddress On-chain address for the agent
   * @param metadataUri IPFS hash or URI pointing to capability declaration
   * @returns Transaction receipt
   */
  registerAgent(
    agentId: string,
    agentAddress: string,
    metadataUri: string,
  ): Promise<TransactionReceipt>;

  /**
   * @notice Update an agent's operational status.
   * @dev Reflects lifecycle transitions on-chain.
   *      Valid transitions: IDLE→RUNNING, RUNNING→PAUSED,
   *      PAUSED→RUNNING, RUNNING→ERROR, ERROR→IDLE (with resolution),
   *      any→INACTIVE (deactivation).
   *
   * @param agentId The agent whose status is changing
   * @param status New status value (enum: 0=IDLE, 1=RUNNING, 2=PAUSED, 3=ERROR)
   * @returns Transaction receipt
   */
  updateAgentStatus(
    agentId: string,
    status: AgentStatus,
  ): Promise<TransactionReceipt>;

  /**
   * @notice Deactivate an agent permanently.
   * @dev Sets active=false. On-chain history is preserved.
   *      Cascades to all sub-agents (parentAgentId chain).
   *      Emits AgentDeactivated event on-chain.
   *
   * @param agentId The agent to deactivate
   * @returns Transaction receipt
   */
  deactivateAgent(agentId: string): Promise<TransactionReceipt>;

  /**
   * @notice Record a significant agent action on-chain.
   * @dev Provides tamper-proof provenance. The actionHash and
   *      resultHash link on-chain records to off-chain detail
   *      stored in PostgreSQL.
   *
   * @param agentId The acting agent
   * @param actionHash Keccak256 hash of the action payload
   * @param resultHash Keccak256 hash of the result payload
   * @param gasUsed Gas consumed by this action
   * @returns Transaction receipt
   */
  recordAgentAction(
    agentId: string,
    actionHash: string,
    resultHash: string,
    gasUsed: bigint,
  ): Promise<TransactionReceipt>;

  /**
   * @notice Retrieve metrics for an agent.
   * @dev Returns cumulative on-chain statistics.
   *
   * @param agentId The agent to query
   * @returns Agent metrics (action count, total gas, avg gas, last action time)
   */
  getAgentMetrics(agentId: string): Promise<AgentMetrics>;

  /**
   * @notice Check whether an agent is registered and active.
   * @dev Used by the integration layer before routing requests.
   *
   * @param agentId The agent to check
   * @returns Registration and active status
   */
  getAgentStatus(agentId: string): Promise<{
    registered: boolean;
    active: boolean;
    status: AgentStatus;
  }>;
}

interface AgentMetrics {
  /** Total number of recorded actions */
  actionCount: number;

  /** Cumulative gas consumed across all actions */
  totalGas: bigint;

  /** Average gas per action */
  averageGas: bigint;

  /** Timestamp of most recent action */
  lastActionTimestamp: number;
}

interface TransactionReceipt {
  /** Transaction hash */
  transactionHash: string;

  /** Block number where the transaction was included */
  blockNumber: number;

  /** Gas used by this transaction */
  gasUsed: bigint;

  /** Whether the transaction succeeded */
  status: boolean;
}
```

### 2.2 Pre-Call Validation

The integration layer validates every OrchestratorFacet call before it reaches the chain. Failed validation never results in an on-chain transaction — it fails fast.

| Operation | Validations |
|-----------|-------------|
| `registerAgent` | `agentId` is unique (not already registered). `agentAddress` is not zero address. `metadataUri` is non-empty. If spawned: scope ⊆ parent scope, budget ≤ parent remaining budget, spawn depth ≤ `maxSpawnDepth`. |
| `updateAgentStatus` | Agent is registered and active. Transition is valid (see AGENTS.md §3.2). ERROR→IDLE requires a resolution record. |
| `deactivateAgent` | Agent is registered. Cascade deactivation list is computed before the call. |
| `recordAgentAction` | Agent is registered and in RUNNING status. Action hash and result hash are non-empty. Gas value is non-zero. |
| `getAgentMetrics` | Agent is registered (active or inactive — metrics survive deactivation). |

### 2.3 Gas Management

OrchestratorFacet operations consume on-chain gas. The integration layer manages gas budgets to prevent agent operations from exhausting system resources.

```typescript
/**
 * @title GasAccountant
 * @notice Tracks gas consumption against budgets.
 * @dev Each agent has a per-action ceiling and a per-workflow ceiling.
 *      The integration layer deducts gas from the agent's budget
 *      after each on-chain operation and rejects operations
 *      that would exceed the ceiling.
 */
interface IGasAccountant {
  /** Check if an agent has sufficient gas budget for an operation */
  canAfford(agentId: string, estimatedGas: bigint): boolean;

  /** Record gas expenditure after a successful transaction */
  recordExpenditure(agentId: string, gasUsed: bigint): void;

  /** Get remaining budget for an agent */
  remainingBudget(agentId: string): {
    perAction: bigint;
    perWorkflow: bigint;
  };

  /** Reset per-workflow budget (called when agent starts a new workflow) */
  resetWorkflowBudget(agentId: string): void;
}
```

---

## 3. KernelFacet Client

The KernelFacet provides high-performance caching (11.1M+ ops/sec benchmarked). B.R.A.N.D.I. uses it for intermediate agent results, shared state between workflow nodes, and hot agent context.

### 3.1 Contract Interface (Consumed)

```typescript
/**
 * @title IKernelFacetClient
 * @notice B.R.A.N.D.I.'s client interface to KernelFacet.
 * @dev All calls require KERNEL_OPERATOR_ROLE.
 *      Values stored on-chain are hashes — full payloads
 *      live in Redis via the NestJS backend.
 */
interface IKernelFacetClient {
  /**
   * @notice Store a cache entry.
   * @dev The key is a namespaced string (see §3.2).
   *      The valueHash is keccak256 of the full value stored off-chain.
   *      TTL governs automatic expiration.
   *
   * @param key Namespaced cache key
   * @param valueHash Hash of the cached value
   * @param ttlSeconds Time-to-live in seconds (0 = no expiry)
   * @returns Transaction receipt
   */
  setCacheEntry(
    key: string,
    valueHash: string,
    ttlSeconds: number,
  ): Promise<TransactionReceipt>;

  /**
   * @notice Retrieve a cache entry.
   * @dev Returns null if key doesn't exist or has expired.
   *      The on-chain hash is verified against the off-chain value
   *      retrieved from Redis. Mismatch triggers a cache invalidation
   *      and a MonitoringFacet alert.
   *
   * @param key Namespaced cache key
   * @returns Cache entry or null
   */
  getCacheEntry(key: string): Promise<CacheEntry | null>;

  /**
   * @notice Invalidate a cache entry.
   * @dev Removes both the on-chain hash and the off-chain value.
   *      Used when state changes make cached results stale.
   *
   * @param key Namespaced cache key
   * @returns Transaction receipt
   */
  invalidateCacheEntry(key: string): Promise<TransactionReceipt>;
}

interface CacheEntry {
  /** The cached value (full payload from Redis) */
  value: Record<string, unknown>;

  /** Hash of the value (verified against on-chain record) */
  valueHash: string;

  /** Remaining TTL in seconds */
  remainingTtl: number;

  /** Timestamp when the entry was created */
  createdAt: number;
}
```

### 3.2 Key Namespace Convention

Cache keys are namespaced to prevent collisions between agents, workflows, and system components.

```
Format: {component}.{scope}.{identifier}

Examples:
  agent.{agentId}.context          — Agent's current working context
  agent.{agentId}.tools            — Agent's resolved tool set
  workflow.{workflowId}.state      — Workflow execution state snapshot
  workflow.{workflowId}.node.{nodeId} — Node-level intermediate result
  convergence.{requestId}.evals    — Collected evaluations for a request
  knowledge.{queryHash}            — Cached S.H.A.N.N.O.N. query result
```

The integration layer enforces this convention. Keys not matching the pattern are rejected before reaching KernelFacet.

### 3.3 Cache Integrity Verification

Every cache read verifies the off-chain value against its on-chain hash. This prevents cache poisoning — if Redis is compromised or a value is corrupted, the hash mismatch is detected and the entry is invalidated.

```
Read flow:
  1. Retrieve valueHash from KernelFacet (on-chain)
  2. Retrieve full value from Redis (off-chain)
  3. Compute keccak256(value)
  4. Compare computed hash with on-chain hash
  5. If match: return value
  6. If mismatch: invalidate entry, log alert to MonitoringFacet, return null
```

---

## 4. OracleFacet Client

The OracleFacet manages B.R.A.N.D.I.'s dual-chain result system. Private client chains handle sensitive agent operations. The shared main chain provides consensus and finality. This maps directly to B.R.A.N.D.I.'s branching execution model: branches execute on private chains, consensus boundaries aggregate to the main chain.

### 4.1 Contract Interface (Consumed)

```typescript
/**
 * @title IOracleFacetClient
 * @notice B.R.A.N.D.I.'s client interface to OracleFacet.
 * @dev All calls require ORACLE_OPERATOR_ROLE.
 *      The dual-chain architecture provides both privacy
 *      (client chains) and finality (main chain).
 */
interface IOracleFacetClient {
  /**
   * @notice Submit a result to a private client chain.
   * @dev Used by agents during workflow execution.
   *      Each workflow branch operates on its own client chain.
   *      Results are hashed — full payloads stored off-chain.
   *
   * @param chainId Identifier for the client chain
   * @param resultHash Keccak256 hash of the result payload
   * @param proof Cryptographic proof linking this result to its computation
   * @returns Transaction receipt on the client chain
   */
  submitClientChainResult(
    chainId: string,
    resultHash: string,
    proof: string,
  ): Promise<TransactionReceipt>;

  /**
   * @notice Aggregate results across client chains to the main chain.
   * @dev Called at consensus boundaries. The Aggregator agent
   *      triggers this after synthesizing parallel branch results.
   *      The aggregation creates a single main-chain record
   *      linking all contributing client chain results.
   *
   * @param aggregationId Unique identifier for this aggregation
   * @param chainIds Client chains whose results are being aggregated
   * @param resultHashes Ordered result hashes from each chain
   * @returns Transaction receipt on the main chain
   */
  aggregateResults(
    aggregationId: string,
    chainIds: string[],
    resultHashes: string[],
  ): Promise<TransactionReceipt>;

  /**
   * @notice Retrieve a result from a client chain.
   * @dev Used to verify results during aggregation and auditing.
   *
   * @param chainId The client chain to query
   * @param resultHash The result hash to look up
   * @returns The on-chain result record or null
   */
  getClientChainResult(
    chainId: string,
    resultHash: string,
  ): Promise<ClientChainResult | null>;

  /**
   * @notice Retrieve an aggregation record from the main chain.
   * @dev Provides the full provenance of a consensus boundary outcome.
   *
   * @param aggregationId The aggregation to look up
   * @returns The aggregation record or null
   */
  getAggregation(aggregationId: string): Promise<AggregationRecord | null>;
}

interface ClientChainResult {
  /** The chain this result lives on */
  chainId: string;

  /** Hash of the result */
  resultHash: string;

  /** Proof linking result to computation */
  proof: string;

  /** Block number on the client chain */
  blockNumber: number;

  /** Timestamp of submission */
  submittedAt: number;
}

interface AggregationRecord {
  /** Unique aggregation identifier */
  aggregationId: string;

  /** Contributing client chains */
  chainIds: string[];

  /** Result hashes from each chain (ordered) */
  resultHashes: string[];

  /** Combined hash of the aggregation */
  aggregatedHash: string;

  /** Block number on the main chain */
  mainChainBlockNumber: number;

  /** Timestamp of aggregation */
  aggregatedAt: number;
}
```

### 4.2 Chain Mapping to Workflows

The integration layer manages the relationship between workflows and client chains.

```typescript
/**
 * @title IChainMapper
 * @notice Maps workflow execution branches to client chains.
 * @dev Each branching node in a workflow spawns execution on
 *      a private client chain. The chain mapper tracks which
 *      workflow branch runs on which chain.
 */
interface IChainMapper {
  /** Create or retrieve a client chain for a workflow branch */
  getOrCreateChain(workflowId: string, branchId: string): Promise<string>;

  /** Retrieve all chains associated with a workflow */
  getChainsForWorkflow(workflowId: string): Promise<ChainMapping[]>;

  /** Mark a chain as finalized (no more results will be submitted) */
  finalizeChain(chainId: string): Promise<void>;
}

interface ChainMapping {
  /** The workflow this chain serves */
  workflowId: string;

  /** The branch within the workflow */
  branchId: string;

  /** The client chain identifier */
  chainId: string;

  /** Whether this chain has been finalized */
  finalized: boolean;

  /** Number of results submitted to this chain */
  resultCount: number;
}
```

---

## 5. MonitoringFacet Client

The MonitoringFacet provides real-time observability across the agent swarm. Sentinel agents are the primary consumers, but the integration layer itself reports health data for all system components.

### 5.1 Contract Interface (Consumed)

```typescript
/**
 * @title IMonitoringFacetClient
 * @notice B.R.A.N.D.I.'s client interface to MonitoringFacet.
 * @dev MONITORING_ROLE is held by the system process.
 *      Sentinel agents submit reports through the integration layer.
 */
interface IMonitoringFacetClient {
  /**
   * @notice Report component health status.
   * @dev Called periodically by the integration layer for each
   *      major component (Agent Runtime, Workflow Engine,
   *      Convergence Module, Integration Layer itself).
   *
   * @param component Component identifier
   * @param health Health status payload
   * @returns Transaction receipt
   */
  reportHealth(
    component: string,
    health: ComponentHealth,
  ): Promise<TransactionReceipt>;

  /**
   * @notice Log a system event.
   * @dev Used for anomalies, alerts, and operational events
   *      that require on-chain recording for auditability.
   *
   * @param event The event to log
   * @returns Transaction receipt
   */
  logEvent(event: SystemEvent): Promise<TransactionReceipt>;

  /**
   * @notice Retrieve health history for a component.
   * @dev Used for trend analysis and anomaly detection by Sentinel agents.
   *
   * @param component Component identifier
   * @param fromTimestamp Start of the query window
   * @param toTimestamp End of the query window
   * @returns Array of health records
   */
  getHealthHistory(
    component: string,
    fromTimestamp: number,
    toTimestamp: number,
  ): Promise<ComponentHealth[]>;

  /**
   * @notice Retrieve recent events by severity.
   * @dev Used by Sentinel agents for pattern detection.
   *
   * @param minSeverity Minimum severity to include
   * @param limit Maximum number of events to return
   * @returns Array of system events
   */
  getRecentEvents(
    minSeverity: EventSeverity,
    limit: number,
  ): Promise<SystemEvent[]>;
}

interface ComponentHealth {
  /** Component identifier */
  component: string;

  /** Overall health status */
  status: 'healthy' | 'degraded' | 'unhealthy' | 'unreachable';

  /** Timestamp of this health check */
  checkedAt: number;

  /** Component-specific metrics */
  metrics: Record<string, number>;

  /** Active alerts for this component */
  activeAlerts: string[];
}

interface SystemEvent {
  /** Unique event identifier */
  eventId: string;

  /** Component that generated the event */
  source: string;

  /** Event severity */
  severity: EventSeverity;

  /** Human-readable event description */
  description: string;

  /** Structured event data */
  data: Record<string, unknown>;

  /** Timestamp */
  timestamp: number;

  /** Related agent ID (if applicable) */
  relatedAgentId: string | null;

  /** Related workflow ID (if applicable) */
  relatedWorkflowId: string | null;
}

enum EventSeverity {
  INFO = 'info',
  WARNING = 'warning',
  ERROR = 'error',
  CRITICAL = 'critical',
}
```

### 5.2 Health Check Schedule

The integration layer reports component health on a fixed schedule. Sentinel agents may request additional checks.

| Component | Default Interval | Metrics Reported |
|-----------|-----------------|------------------|
| Agent Runtime | 30s | Active agents, agents in ERROR, avg loop latency |
| Workflow Engine | 30s | Active workflows, pending nodes, recursion depth distribution |
| Convergence Module | 60s | Pending requests, avg convergence time, reversion count |
| NIKO Integration Layer | 15s | Request queue depth, avg response latency, error rate |
| Cache (KernelFacet) | 15s | Hit rate, miss rate, integrity violations |
| Chain (OracleFacet) | 60s | Active client chains, pending aggregations, main chain lag |

### 5.3 Alert Escalation

Events at different severity levels trigger different responses.

| Severity | Integration Layer Response |
|----------|--------------------------|
| INFO | Log to off-chain store. No on-chain recording. |
| WARNING | Log to off-chain store. Record on-chain if pattern persists (3+ in 5 minutes). |
| ERROR | Record on-chain immediately. Notify relevant Sentinel agents via Event Bus. |
| CRITICAL | Record on-chain immediately. Trigger convergence request (the system needs Admin/Counsel/Human to evaluate the threat). |

---

## 6. TreasuryFacet Client

The TreasuryFacet manages resource allocation — the economic substrate of the swarm. B.R.A.N.D.I. agents consume resources (gas, compute, external API calls). The TreasuryFacet ensures these resources are allocated according to the collusive resource allocation model described in the cooperative trading paper.

### 6.1 Contract Interface (Consumed)

```typescript
/**
 * @title ITreasuryFacetClient
 * @notice B.R.A.N.D.I.'s client interface to TreasuryFacet.
 * @dev Multi-party payment distribution for resource allocation.
 *      The treasury is the economic expression of iterative mutualism —
 *      resources flow to where they produce the most mutual benefit.
 */
interface ITreasuryFacetClient {
  /**
   * @notice Query the balance available to a given address.
   * @dev Used to check resource availability before allocation.
   *
   * @param address The address to query
   * @returns Balance in the treasury's unit of account
   */
  getBalance(address: string): Promise<bigint>;

  /**
   * @notice Request a resource allocation for a specific purpose.
   * @dev The purpose string is recorded for provenance.
   *      Allocation requests that exceed configured thresholds
   *      require convergence (enforced by the integration layer,
   *      not by the facet itself).
   *
   * @param amount Amount to allocate
   * @param purpose Human-readable purpose description
   * @param recipientAddress Address to receive the allocation
   * @returns Whether the allocation was approved
   */
  requestAllocation(
    amount: bigint,
    purpose: string,
    recipientAddress: string,
  ): Promise<AllocationResult>;

  /**
   * @notice Record a resource expenditure.
   * @dev Called after an agent consumes allocated resources.
   *      Links expenditure to the originating allocation.
   *
   * @param allocationId The allocation being drawn from
   * @param amount Amount consumed
   * @param agentId The consuming agent
   * @returns Transaction receipt
   */
  recordExpenditure(
    allocationId: string,
    amount: bigint,
    agentId: string,
  ): Promise<TransactionReceipt>;

  /**
   * @notice Get allocation history for auditing.
   * @dev Full provenance of resource flow through the system.
   *
   * @param fromTimestamp Start of query window
   * @param toTimestamp End of query window
   * @returns Array of allocation records
   */
  getAllocationHistory(
    fromTimestamp: number,
    toTimestamp: number,
  ): Promise<AllocationRecord[]>;
}

interface AllocationResult {
  /** Whether the allocation was approved */
  approved: boolean;

  /** Allocation identifier (for tracking expenditures) */
  allocationId: string | null;

  /** If rejected, the reason */
  rejectionReason: string | null;
}

interface AllocationRecord {
  /** Unique allocation identifier */
  allocationId: string;

  /** Amount allocated */
  amount: bigint;

  /** Amount consumed so far */
  consumed: bigint;

  /** Purpose description */
  purpose: string;

  /** Recipient address */
  recipientAddress: string;

  /** Agent that consumed the resources (if applicable) */
  consumingAgentId: string | null;

  /** Timestamp of allocation */
  allocatedAt: number;

  /** Whether this allocation required convergence */
  requiredConvergence: boolean;
}
```

### 6.2 Allocation Thresholds

Resource allocations above certain thresholds require convergence. These thresholds are configurable through the convergence model but have sensible defaults.

```typescript
/**
 * @title AllocationThresholds
 * @notice Defines when resource allocations require convergence.
 * @dev These values are configurable — changes require convergence.
 *      Defaults are conservative to prevent resource drain.
 */
interface AllocationThresholds {
  /** Single allocation above this amount requires convergence */
  singleAllocationCeiling: bigint;

  /** Cumulative allocation within a time window above this requires convergence */
  windowAllocationCeiling: bigint;

  /** Time window for cumulative tracking (seconds) */
  windowDurationSeconds: number;

  /** Allocation to an address not previously allocated to requires convergence */
  newRecipientRequiresConvergence: boolean;
}
```

---

## 7. Backend API Endpoints

N.I.K.O.System exposes a NestJS backend with REST, GraphQL, and WebSocket APIs. The integration layer consumes these for off-chain operations that complement the on-chain facet interactions.

### 7.1 REST Endpoints

| Method | Path | Purpose | Facet Complement |
|--------|------|---------|-----------------|
| POST | `/agents/register` | Register agent (off-chain metadata storage) | OrchestratorFacet.registerAgent |
| PATCH | `/agents/{agentId}/status` | Update agent status (off-chain state) | OrchestratorFacet.updateAgentStatus |
| POST | `/agents/{agentId}/actions` | Record agent action (off-chain detail) | OrchestratorFacet.recordAgentAction |
| GET | `/agents/{agentId}/metrics` | Retrieve agent metrics | OrchestratorFacet.getAgentMetrics |
| GET | `/agents/{agentId}/state` | Retrieve agent hot state from Redis | KernelFacet cache |
| POST | `/workflows` | Create workflow definition | PostgreSQL storage |
| GET | `/workflows/{workflowId}/status` | Query workflow execution status | PostgreSQL + Redis |
| POST | `/convergence/requests` | Submit convergence request | OracleFacet (hash recording) |
| GET | `/convergence/{requestId}/directive` | Poll for convergence directive | — |
| POST | `/reports` | Submit agent report (off-chain detail) | OrchestratorFacet.recordAgentAction |
| POST | `/knowledge/query` | Query S.H.A.N.N.O.N. knowledge | Routed through to S.H.A.N.N.O.N. |
| GET | `/monitoring/health/{component}` | Get component health | MonitoringFacet.reportHealth |
| GET | `/monitoring/events` | Get recent system events | MonitoringFacet |
| POST | `/treasury/allocate` | Request resource allocation | TreasuryFacet.requestAllocation |
| GET | `/treasury/balance/{address}` | Query balance | TreasuryFacet.getBalance |

### 7.2 GraphQL Schema (Knowledge Queries)

Knowledge queries to S.H.A.N.N.O.N. use GraphQL for structured retrieval.

```graphql
type Query {
  """Query S.H.A.N.N.O.N. for constitutionally curated knowledge."""
  knowledge(input: KnowledgeQueryInput!): CuratedKnowledgeResponse!

  """Retrieve agent state and history."""
  agent(agentId: ID!): AgentDetail

  """Retrieve workflow execution state."""
  workflow(workflowId: ID!): WorkflowDetail

  """Retrieve convergence request status."""
  convergence(requestId: ID!): ConvergenceDetail
}

input KnowledgeQueryInput {
  """The agent requesting knowledge"""
  agentId: ID!

  """Natural language query"""
  query: String!

  """Domain context to focus retrieval"""
  domain: String!

  """Maximum number of result chunks"""
  maxResults: Int = 5
}

type CuratedKnowledgeResponse {
  """Unique query identifier"""
  queryId: ID!

  """Retrieved knowledge chunks, constitutionally curated"""
  chunks: [KnowledgeChunk!]!

  """Constitutional hash at time of curation"""
  constitutionalHash: String!

  """Timestamp of retrieval"""
  retrievedAt: Int!
}

type KnowledgeChunk {
  """Content of the knowledge chunk"""
  content: String!

  """Source metadata"""
  source: String!

  """Relevance score from vector similarity"""
  relevanceScore: Float!

  """Domain classification"""
  domain: String!
}
```

### 7.3 WebSocket Channels

WebSocket connections provide real-time communication for events that cannot tolerate polling latency.

| Channel | Direction | Payload | Use Case |
|---------|-----------|---------|----------|
| `ws://niko/agents/{agentId}/status` | Server → Client | `AgentStatus` updates | Agent Runtime monitors agent lifecycle |
| `ws://niko/workflows/{workflowId}/progress` | Server → Client | Node completion events | Workflow Engine tracks execution |
| `ws://niko/convergence/{requestId}/directive` | Server → Client | `ConvergenceDirective` or `MutualismContext` | Agent blocks here during report-and-wait |
| `ws://niko/events` | Server → Client | `BrandiEventType` events | Sentinel agents and system monitors |

The convergence WebSocket channel (`/convergence/{requestId}/directive`) is the critical path for the report-and-wait pattern. When an agent hits a boundary, it submits a convergence request via REST, then opens a WebSocket on this channel and blocks. The directive arrives through this channel when convergence manifests — or a `MutualismContext` arrives if the system reverts to iterative mutualism.

---

## 8. Event Subscriptions

B.R.A.N.D.I. both publishes and subscribes to events on N.I.K.O.System's RxJS Event Bus.

### 8.1 Events Published by B.R.A.N.D.I.

These events are published by the integration layer after successful facet operations.

| Event | Trigger | Payload |
|-------|---------|---------|
| `brandi.agent.registered` | Agent registration confirmed on-chain | `{ agentId, agentType, parentAgentId }` |
| `brandi.agent.status_changed` | Agent status transition confirmed | `{ agentId, previousStatus, newStatus }` |
| `brandi.agent.deactivated` | Agent deactivation confirmed | `{ agentId, cascadedAgentIds }` |
| `brandi.agent.boundary_hit` | Agent boundary detection triggered | `{ agentId, boundaryReason, convergenceRequestId }` |
| `brandi.workflow.started` | Workflow execution initiated | `{ workflowId, triggerId, triggerType }` |
| `brandi.workflow.completed` | Workflow execution finished | `{ workflowId, resultHash, duration }` |
| `brandi.workflow.failed` | Workflow execution failed | `{ workflowId, errorDescription, failedNodeId }` |
| `brandi.workflow.branched` | Workflow forked into parallel paths | `{ workflowId, branchNodeId, branchCount, chainIds }` |
| `brandi.workflow.consensus_boundary` | Consensus boundary reached | `{ workflowId, boundaryNodeId, aggregationId }` |
| `brandi.convergence.requested` | Convergence request submitted | `{ requestId, originAgentId, impactLevel }` |
| `brandi.convergence.achieved` | Convergence manifested | `{ requestId, directiveHash }` |
| `brandi.convergence.reverted` | Non-convergence, reverted to mutualism | `{ requestId, firstLogisticalReality }` |
| `brandi.report.submitted` | Agent report recorded | `{ agentId, reportOutcome, actionHash }` |

### 8.2 Events Consumed by B.R.A.N.D.I.

These events originate from N.I.K.O.System infrastructure and trigger B.R.A.N.D.I. responses.

| Event | Source | B.R.A.N.D.I. Response |
|-------|--------|----------------------|
| `niko.chain.block_finalized` | Consensus layer | Workflow Engine checks for pending chain-event triggers |
| `niko.chain.agent_action_recorded` | OrchestratorFacet | Integration layer updates off-chain state to match on-chain confirmation |
| `niko.chain.aggregation_complete` | OracleFacet | Workflow Engine advances past the associated consensus boundary |
| `niko.governance.upgrade_proposed` | GovernanceFacet | Convergence request generated (system must evaluate the upgrade) |
| `niko.monitoring.threshold_breach` | MonitoringFacet | Sentinel agents evaluate the breach, may trigger convergence |

### 8.3 Event Reliability

Events are the nervous system of the integration. Lost events cause state divergence. The integration layer enforces the following guarantees:

1. **At-least-once delivery.** Events are persisted to PostgreSQL before publication. If a subscriber does not acknowledge, the event is re-delivered.
2. **Idempotency.** Every event carries a unique `eventId`. Handlers must be idempotent — processing the same event twice produces the same result.
3. **Ordering within scope.** Events for a single agent or workflow are delivered in order. Cross-agent event ordering is not guaranteed and must not be relied upon.
4. **Dead letter queue.** Events that fail processing after configurable retry attempts are moved to a dead letter queue for manual investigation. Dead-lettered events trigger a WARNING-level MonitoringFacet alert.

---

## 9. Error Handling

### 9.1 Error Categories

| Category | Example | Integration Layer Response |
|----------|---------|---------------------------|
| **Transient network** | RPC timeout, WebSocket disconnect | Retry with exponential backoff (max 5 attempts). If all fail, agent enters ERROR status. |
| **On-chain revert** | Insufficient gas, role check failure | Log the revert reason. Do not retry (reverts are deterministic). Agent enters ERROR status with revert data. |
| **Validation failure** | Invalid scope subset, duplicate agentId | Reject immediately. No on-chain transaction. Return structured error to caller. |
| **State inconsistency** | Off-chain state diverges from on-chain | Trigger reconciliation: read on-chain state, overwrite off-chain. Log CRITICAL event. |
| **S.H.A.N.N.O.N. unreachable** | Knowledge query timeout, report delivery failure | Cache the request. Retry when connectivity is restored. Agent may continue if knowledge was non-critical, or enter report-and-wait if knowledge was required for the current action. |

### 9.2 Reconciliation Protocol

When the integration layer detects state inconsistency between on-chain and off-chain records, it runs reconciliation:

```
1. Halt new operations for the affected scope (agent or workflow)
2. Read authoritative on-chain state from OrchestratorFacet
3. Compare with off-chain state in PostgreSQL/Redis
4. Overwrite off-chain state with on-chain truth
5. Log CRITICAL event to MonitoringFacet
6. Resume operations
7. If reconciliation fails, trigger convergence
```

On-chain state is always authoritative. Off-chain state is always subordinate. This is non-negotiable.

---

## 10. Security Considerations

### 10.1 Role Isolation

The integration layer holds system-level roles. It never exposes these roles to agents or external callers. The role boundary is enforced at the integration layer, not at the agent level.

```
Agent request → Integration Layer validates scope → Integration Layer calls facet with system role
                                                     ↑
                                              Agent NEVER touches this
```

### 10.2 Diamond Pattern Risks (Mitigations)

Per the project's contract knowledge base, EIP-2535 introduces specific risks that the integration layer must account for:

| Risk | Mitigation |
|------|-----------|
| **Function selector clash** | The integration layer maintains a selector registry mapping every known facet function. New facets are validated against this registry before the system accepts them. Clashes block the upgrade. |
| **Storage layout collision** | All storage layouts are managed through Diamond Storage pattern (per-facet storage structs at deterministic slots). The integration layer does not manage storage directly but verifies storage consistency during post-upgrade health checks. |
| **Increased attack surface** | Each facet addition or replacement triggers a GovernanceFacet upgrade proposal, which triggers a convergence request. No facet change occurs without Admin/Counsel/Human alignment. |

### 10.3 Transport Security

| Channel | Security |
|---------|----------|
| REST API | HTTPS with mutual TLS. API key authentication. |
| GraphQL | Same as REST. Query depth limiting to prevent abuse. |
| WebSocket | WSS (TLS). Connection authenticated at handshake. Heartbeat every 30s. |
| On-chain | Transactions signed by system wallet. Multi-sig for high-value operations. |

---

## 11. Implementation Guidance

### 11.1 Build Order

The integration layer is the second priority in the implementation roadmap (after shared types). Build in this order:

```
1. TransactionReceipt + error types (shared types)
   └── 2. IOrchestratorFacetClient (agent lifecycle is foundational)
       └── 3. IGasAccountant (budget tracking before any other operations)
           └── 4. IKernelFacetClient (caching needed by all components)
               └── 5. IOracleFacetClient (chain operations needed by workflows)
                   └── 6. IMonitoringFacetClient (observability for everything above)
                       └── 7. ITreasuryFacetClient (resource allocation)
                           └── 8. INikoClient (composite facade wrapping 2-7)
                               └── 9. S.H.A.N.N.O.N. routing (knowledge, convergence, reports)
                                   └── 10. Event Bus integration (subscriptions + publications)
```

### 11.2 Testing Strategy

| Layer | Test Type | What to Verify |
|-------|-----------|----------------|
| Facet clients | Unit (mocked chain) | Every facet call succeeds/fails correctly. Validation catches bad input. |
| Gas accountant | Unit | Budget tracking is accurate. Ceiling enforcement works. Reset works. |
| Cache integrity | Unit | Hash verification detects tampering. Invalidation cascades correctly. |
| Chain mapper | Unit | Branch-to-chain mapping is consistent. Finalization prevents further submissions. |
| Event publication | Integration | Every facet operation publishes the correct event. Events carry correct payloads. |
| Event consumption | Integration | Every consumed event triggers the correct B.R.A.N.D.I. response. |
| Reconciliation | Integration | Deliberate state divergence is detected and corrected. |
| End-to-end | System | Agent registration → action → report flow completes with correct on-chain and off-chain state. |

### 11.3 File Headers

Every source file in the integration layer must include:

```typescript
/**
 * @file [filename]
 * @title [component name]
 * @notice [what this file does]
 * @dev Part of the NIKO Integration Layer — B.R.A.N.D.I.'s single
 *      interface to N.I.K.O.System infrastructure. No other component
 *      may access N.I.K.O.System directly.
 *
 * @copyright 2024 John A. Welch, Director, Blinded Eye Foundation.
 * All rights reserved.
 * @license AGPL-3.0
 */
```

---

**Copyright © 2024 John A. Welch, Director, Blinded Eye Foundation. All rights reserved.**
