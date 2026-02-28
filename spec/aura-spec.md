# AURA Specification v0.1.0

## Abstract

AURA is a metrics framework that measures the reliability and performance of AI agent output. Like DORA measures software delivery performance through four key metrics, AURA measures agent performance through four: Feature Frequency, Feature Lead Time, Human Intervention Rate, and Recovery Efficiency. AURA provides a common language for evaluating whether an AI agent is producing work autonomously, quickly, and consistently.

## Status

Draft — seeking feedback.

## 1. Terminology

- **Deliverable**: A unit of agent work verified against a specification. One deliverable = one measurable outcome. Examples: a feature implementation, a bug fix, a refactoring task.

- **Spec**: The acceptance criteria for a deliverable. Can come from any source — an OpenSpec change folder, a Jira ticket, a markdown file, a plain text prompt. The spec defines what "done" means.

- **Phase**: A stage in the deliverable lifecycle. The canonical phases are: `propose`, `specs`, `design`, `tasks`, `apply`, `verify`, `archive`. Not all phases are required for every deliverable.

- **Conformance**: The degree to which a deliverable meets its spec. Used in the verify phase to determine acceptance or failure.

- **Recovery**: A rework cycle triggered by a failure during execution. When verification fails and the agent retries, that retry is a recovery attempt.

## 2. The Four Metrics

### 2.1 Feature Frequency

- **Definition**: The number of deliverables accepted per unit time.
- **Calculation**: `count(deliverables where status = "accepted") / time_period`
- **Unit**: deliverables/day (or /week)
- **Performance tiers**:

| Tier | Threshold |
|------|-----------|
| Elite | ≥3/day |
| High | ≥1/day |
| Medium | ≥1/week |
| Low | <1/week |

- **Why it matters**: Measures an agent's productive capacity at the deliverable level.

### 2.2 Feature Lead Time

- **Definition**: Wall-clock time from spec received to deliverable accepted.
- **Calculation**: `accepted_at - started_at` (in seconds)
- **Unit**: seconds
- **Performance tiers**:

| Tier | Threshold |
|------|-----------|
| Elite | <3,600s (1 hour) |
| High | <14,400s (4 hours) |
| Medium | <86,400s (1 day) |
| Low | ≥86,400s |

- **Why it matters**: Measures end-to-end speed including planning, execution, and verification.
- **Note**: Includes human review time. To isolate agent time, subtract human wait phases from the total.

### 2.3 Human Intervention Rate

- **Definition**: Percentage of deliverables that required human takeover or were abandoned without completion.
- **Calculation**: `count(human_takeover or abandoned) / count(deliverables) × 100`
- **Unit**: percentage
- **Performance tiers**:

| Tier | Threshold |
|------|-----------|
| Elite | <5% |
| High | <10% |
| Medium | <15% |
| Low | ≥15% |

- **Why it matters**: Measures autonomy. A low rate means the agent completes work without requiring human intervention.
- **Note**: A deliverable that required agent self-correction (recovery) but completed without human involvement is NOT counted. Only deliverables where a human stepped in or the work was abandoned are counted.

### 2.4 Recovery Efficiency

- **Definition**: The proportion of total effort spent on rework and retries.
- **Calculation**: `recovery_time / total_time × 100` or `recovery_tool_calls / total_tool_calls × 100`
- **Unit**: percentage
- **Performance tiers**:

| Tier | Threshold |
|------|-----------|
| Elite | <5% |
| High | <10% |
| Medium | <20% |
| Low | ≥20% |

- **Why it matters**: Measures how cleanly an agent executes. High recovery overhead suggests the agent is guessing rather than planning.

## 3. Deliverable Lifecycle

A deliverable progresses through phases:

```
propose → specs → design → tasks → apply → verify → archive
```

- Not all phases are required. A minimal deliverable has: start → apply → archive.
- Phases can repeat (apply → verify → apply → verify → archive).
- Phase transitions are detected by tooling, not mandated by the spec.

### 3.1 Phase Definitions

| Phase | Description | Data Captured |
|-------|-------------|---------------|
| `propose` | Deliverable is identified and scoped | Start timestamp, deliverable ID, description |
| `specs` | Requirements and acceptance criteria are defined | Start/end timestamps, requirements count |
| `design` | Solution approach is planned | Start/end timestamps, design artifacts |
| `tasks` | Work is broken into discrete tasks | Start/end timestamps, task count |
| `apply` | Agent executes the work | Start/end timestamps, tool calls, files changed |
| `verify` | Output is validated against the spec | Start/end timestamps, test results, conformance score |
| `archive` | Deliverable is finalized and recorded | End timestamp, final metrics |

### 3.2 Deliverable States

A deliverable is always in one of these states:

| State | Description |
|-------|-------------|
| `planning` | In propose, specs, design, or tasks phase |
| `executing` | In apply phase |
| `verifying` | In verify phase |
| `completed` | Archived successfully |
| `failed` | Abandoned or below conformance threshold |

## 4. Failure Taxonomy

When a deliverable fails, it should be classified by failure type:

| Failure Type | Description |
|--------------|-------------|
| `spec_misunderstanding` | Agent misinterpreted requirements |
| `hallucination` | Agent produced fabricated output |
| `infinite_loop` | Agent got stuck in a retry cycle |
| `tool_failure` | External tool returned an error the agent couldn't recover from |
| `constraint_violation` | Output exceeded specified boundaries |
| `incomplete` | Agent stopped before finishing all requirements |
| `regression` | Agent's fix broke something that previously worked |

## 5. Data Model

AURA defines three JSON Schema documents. All schemas use JSON Schema draft 2020-12.

### 5.1 Deliverable State (in-progress tracking)

Reference: [`schemas/latest/deliverable-state.schema.json`](../schemas/latest/deliverable-state.schema.json)

Tracks the current state of an in-progress deliverable. Updated as the agent progresses through phases.

| Field | Type | Required | Description | Example |
|-------|------|----------|-------------|---------|
| `change_id` | string | Yes | Unique identifier for the deliverable | `"add-dark-mode"` |
| `status` | enum | Yes | Current state | `"executing"` |
| `started_at` | date-time | Yes | When the deliverable began | `"2026-02-26T10:00:00Z"` |
| `updated_at` | date-time | No | Last update timestamp | `"2026-02-26T10:30:00Z"` |
| `complexity` | enum | No | Estimated complexity | `"moderate"` |
| `description` | string | No | Human-readable summary | `"Add dark mode toggle"` |
| `spec_source` | object | No | Where the spec came from | See below |
| `spec_source.framework` | string | No | Spec framework name | `"openspec"` |
| `spec_source.spec_id` | string | No | Identifier within the framework | `"changes/add-dark-mode"` |
| `spec_source.requirements_count` | integer | No | Number of requirements | `5` |
| `phases` | object | No | Phase timing records | See below |
| `phases.<name>.started_at` | date-time | No | When the phase started | `"2026-02-26T10:05:00Z"` |
| `phases.<name>.completed_at` | date-time | No | When the phase ended | `"2026-02-26T10:10:00Z"` |
| `current_phase` | string | No | Active phase name | `"apply"` |
| `tasks_total` | integer | No | Total planned tasks | `8` |
| `tasks_completed` | integer | No | Tasks finished so far | `4` |
| `tool_calls` | object | No | Tool call counts by tool name | `{"file_edit": 12, "bash": 5}` |
| `tool_calls_total` | integer | No | Total tool calls across all tools | `17` |
| `recovery_attempts` | integer | No | Number of recovery cycles | `1` |
| `apply_iterations` | integer | No | Number of apply cycles | `2` |
| `sessions` | array | No | Session identifiers | `["session-1", "session-2"]` |

### 5.2 Metrics Output (completed deliverable)

Reference: [`schemas/latest/metrics-output.schema.json`](../schemas/latest/metrics-output.schema.json)

The final record emitted when a deliverable is completed or failed. This is the primary AURA output format.

| Field | Type | Required | Description | Example |
|-------|------|----------|-------------|---------|
| `schema_version` | string | Yes | AURA schema version | `"0.1.0"` |
| `change_id` | string | Yes | Unique deliverable identifier | `"add-dark-mode"` |
| `started_at` | date-time | Yes | When the deliverable began | `"2026-02-26T10:00:00Z"` |
| `completed_at` | date-time | Yes | When it finished | `"2026-02-26T10:45:00Z"` |
| `status` | enum | Yes | Final state | `"completed"` |
| `description` | string | No | Human-readable summary | `"Add dark mode toggle"` |
| `metrics` | object | Yes | The measured metrics | See below |
| `metrics.resolution_latency_seconds` | number | No | Total wall-clock seconds | `2700` |
| `metrics.phase_durations` | object | No | Seconds per phase | `{"apply": 1800, "verify": 300}` |
| `metrics.tool_calls` | object | No | Tool call counts | `{"file_edit": 24, "total": 45}` |
| `metrics.apply_iterations` | integer | No | Apply cycle count | `2` |
| `metrics.recovery_attempts` | integer | No | Recovery cycle count | `1` |
| `metrics.tasks_completed` | integer | No | Tasks finished | `12` |
| `metrics.tasks_total` | integer | No | Tasks planned | `12` |
| `metrics.deliverable_failed` | boolean | No | Whether the deliverable failed | `false` |
| `metrics.failure_type` | string | No | Failure classification | `null` |
| `spec_source` | object | No | Where the spec came from | See 5.1 |
| `agent` | object | No | Agent information | See below |
| `agent.name` | string | No | Agent identifier | `"claude-code"` |
| `agent.model` | string | No | Model used | `"claude-sonnet-4-20250514"` |
| `agent.framework` | string | No | Agent framework | `"claude-code"` |
| `sessions` | array | No | Session identifiers | `["session-abc"]` |

### 5.3 AURA Event (individual phase record)

Reference: [`schemas/latest/aura-event.schema.json`](../schemas/latest/aura-event.schema.json)

Lightweight event records for streaming and real-time collection. Emitted at phase transitions and significant moments during execution.

| Field | Type | Required | Description | Example |
|-------|------|----------|-------------|---------|
| `event_type` | enum | Yes | Type of event | `"phase_start"` |
| `timestamp` | date-time | Yes | When the event occurred | `"2026-02-26T10:05:00Z"` |
| `change_id` | string | Yes | Deliverable identifier | `"add-dark-mode"` |
| `phase` | string | No | Phase name (if applicable) | `"apply"` |
| `data` | object | No | Event-specific payload | `{"task_index": 3}` |

Event types: `phase_start`, `phase_end`, `tool_call`, `recovery`, `deliverable_start`, `deliverable_end`.

## 6. OpenTelemetry Integration

AURA is designed to work with OpenTelemetry. Instrumented agents emit AURA data as OTel spans and attributes, enabling integration with existing observability infrastructure.

### 6.1 Namespace

All AURA attributes use the `aura.*` prefix. This avoids collision with existing OTel semantic conventions and clearly identifies AURA-specific data.

### 6.2 Span Hierarchy

```
aura.deliverable (root span)
├── aura.deliverable.plan
├── aura.deliverable.execute
│   ├── aura.task
│   │   ├── gen_ai.chat (OTel GenAI standard)
│   │   └── aura.tool.call
│   └── aura.recovery.attempt
├── aura.deliverable.validate
└── aura.deliverable.accept
```

The root `aura.deliverable` span covers the entire deliverable lifecycle. Child spans correspond to phases and sub-operations.

### 6.3 Semantic Attributes

| Attribute | Type | Description | Example |
|-----------|------|-------------|---------|
| `aura.deliverable.id` | string | Unique deliverable identifier | `"add-dark-mode"` |
| `aura.deliverable.type` | string | Deliverable type | `"feature"` |
| `aura.deliverable.status` | string | Current or final status | `"completed"` |
| `aura.deliverable.complexity` | string | Estimated complexity | `"moderate"` |
| `aura.spec.framework` | string | Spec source framework | `"openspec"` |
| `aura.spec.id` | string | Spec identifier | `"changes/add-dark-mode"` |
| `aura.spec.requirements_count` | int | Number of requirements | `5` |
| `aura.phase.name` | string | Current phase | `"apply"` |
| `aura.phase.iteration` | int | Phase iteration count | `2` |
| `aura.failure.type` | string | Failure classification | `"infinite_loop"` |
| `aura.recovery.attempt` | int | Current recovery attempt number | `2` |
| `aura.recovery.total` | int | Total recovery attempts | `3` |
| `aura.tokens.input` | int | Total input tokens consumed | `12000` |
| `aura.tokens.output` | int | Total output tokens generated | `8000` |
| `aura.tokens.total` | int | Total tokens (input + output) | `20000` |
| `aura.tokens.estimated_cost_usd` | double | Estimated cost in USD | `0.12` |
| `aura.human.interventions` | int | Number of human interventions during execution | `0` |
| `aura.agent.name` | string | Agent identifier | `"claude-code"` |
| `aura.agent.model` | string | Model used | `"claude-sonnet-4-20250514"` |
| `aura.agent.framework` | string | Agent framework | `"claude-code"` |

Reference: [`otel/semantic-conventions.yaml`](../otel/semantic-conventions.yaml)

### 6.4 Metric Instruments

| Instrument | Type | Unit | Description |
|------------|------|------|-------------|
| `aura.deliverables.count` | Counter | `{deliverable}` | Total deliverables processed |
| `aura.deliverables.accepted` | Counter | `{deliverable}` | Deliverables accepted |
| `aura.deliverables.failed` | Counter | `{deliverable}` | Deliverables failed |
| `aura.resolution_latency` | Histogram | `s` | Resolution latency distribution |
| `aura.recovery.attempts` | Histogram | `{attempt}` | Recovery attempts per deliverable |
| `aura.tool_calls.count` | Counter | `{call}` | Total tool calls |

Labels: `aura.deliverable.type`, `aura.agent.name`, `aura.failure.type`.

## 7. Spec Framework Compatibility

AURA is spec-framework agnostic. Any system that defines acceptance criteria for agent work can be an AURA spec source.

| Source | Spec = | Requirements = | Deliverable boundary = |
|--------|--------|----------------|----------------------|
| OpenSpec | Change folder | Delta spec requirements | propose → archive |
| Jira | Ticket | Acceptance criteria | Ticket created → Done |
| GitHub Issue | Issue body | Checklist items | Issue opened → closed |
| Plain markdown | The markdown file | Bullet points | File created → verified |
| User prompt | The prompt text | Implicit (single requirement) | Prompt → accepted output |

The `spec_source` field in the data model captures which framework was used. Integrations are responsible for mapping their native structures to AURA concepts.

## 8. Supporting Metrics

The four headline metrics tell you *what's happening*. The supporting metrics tell you *why*. Every supporting metric feeds at least one headline metric. When a headline metric goes red on a dashboard, drill into the supporting metrics to find the cause.

### 8.1 Token Usage

- **Definition**: Total input tokens, output tokens, and cost per deliverable.
- **Feeds**: Recovery Efficiency, Feature Frequency
- **Why it matters**: If an agent uses 50k tokens on a deliverable that should take 10k, the extra 40k is recovery overhead. It also contextualises Feature Frequency — are you shipping more because the agent is efficient, or because it's brute-forcing with expensive models? Token cost per deliverable is the unit economics metric teams will care about most once they're past the "does it work" phase.

### 8.2 Tool Call Count

- **Definition**: Tool call counts by type (Write, Edit, Bash, Read, etc.).
- **Feeds**: Recovery Efficiency, Feature Lead Time
- **Why it matters**: A healthy deliverable has a predictable ratio of reads to writes. If you see 40 Read calls and 2 Writes, the agent spent most of its time searching. If you see 15 Write calls and 12 Edit calls on the same files, it's rewriting its own work. The pattern tells you where the agent is struggling.

### 8.3 Phase Duration

- **Definition**: Time spent in each phase (propose, specs, design, tasks, apply, verify).
- **Feeds**: Feature Lead Time
- **Why it matters**: Decomposes Feature Lead Time into its parts. A 4-hour deliverable where 3.5 hours was in `apply` is very different from one where 2 hours was human review during `verify`. This is where you find the bottleneck — is the agent slow, or is the human slow to review?

### 8.4 Apply Iterations

- **Definition**: How many times the apply phase ran before acceptance.
- **Feeds**: Recovery Efficiency
- **Why it matters**: First-time-right deliverables have zero recovery overhead. Tracking the count over time tells you if your agent is getting better or worse at completing work without rework.

### 8.5 Recovery Attempts

- **Definition**: Count of rework cycles within an apply iteration.
- **Feeds**: Recovery Efficiency
- **Why it matters**: Different from apply iterations. An apply iteration is "the agent stopped, human said try again." A recovery attempt is "the agent hit an error and self-corrected within a single run." High recovery attempts with eventual success means the agent is resilient but inefficient. High recovery attempts with failure means it's thrashing.

### 8.6 Failure Type Distribution

- **Definition**: Breakdown across failure types (spec_misunderstanding, hallucination, infinite_loop, tool_failure, constraint_violation, incomplete, regression).
- **Feeds**: Human Intervention Rate
- **Why it matters**: Categorises *why* humans had to intervene or deliverables were abandoned. A 12% intervention rate means very different things if it's all tool failures (infrastructure problem) versus all hallucinations (model problem). This is the metric that tells you where to invest.

### 8.7 Human Intervention Count

- **Definition**: Number of times a human had to step in during execution.
- **Feeds**: Human Intervention Rate, Recovery Efficiency
- **Why it matters**: An agent that completes every deliverable but needs 5 human nudges per run isn't really autonomous. This metric tracks the journey toward full autonomy.

### 8.8 Complexity Distribution

- **Definition**: Trivial/simple/moderate/complex breakdown of deliverables.
- **Feeds**: Feature Frequency, Feature Lead Time
- **Why it matters**: Contextualises Feature Frequency and Feature Lead Time. Shipping 10 trivial deliverables/day is not the same as shipping 2 complex ones. Without this, you can game throughput by splitting work into tiny pieces.

### 8.10 Relationship Map

```
Token Usage ──────────┐
Tool Call Count ──────┼──→ Recovery Efficiency
Recovery Attempts ────┘
                      ↕
Phase Duration ───────────→ Feature Lead Time
                      ↕
Failure Type ─────────┐
Human Interventions ──┴──→ Human Intervention Rate
                      ↕
Complexity Distribution ──→ Feature Frequency
```

## 9. Performance Tiers

Summary of all performance tiers across the four metrics:

| Metric | Elite | High | Medium | Low |
|--------|-------|------|--------|-----|
| Feature Frequency | ≥3/day | ≥1/day | ≥1/week | <1/week |
| Feature Lead Time | <1 hour | <4 hours | <1 day | ≥1 day |
| Human Intervention Rate | <5% | <10% | <15% | ≥15% |
| Recovery Efficiency | <5% | <10% | <20% | ≥20% |

Tier classification uses the most recent rolling window. The recommended default window is 7 days or 20 deliverables, whichever comes first.

## 10. Philosophy

1. **Measure deliverables, not tasks.** A task is a means to an end. The deliverable — a verified outcome against a spec — is what matters.

2. **Speed, stability, and quality correlate.** Teams and agents that deliver quickly tend to also deliver reliably. AURA measures all three and expects them to move together.

3. **The spec is the source of truth.** Without a spec, there is no conformance. AURA incentivizes clear specifications because measurement requires them.

4. **Metrics drive conversations, not punishments.** AURA metrics identify where to improve, not who to blame. A low score is a signal, not a verdict.

5. **Build on existing standards.** AURA extends OpenTelemetry rather than inventing its own telemetry layer. It works with existing spec frameworks rather than replacing them.

6. **Start simple, go deep.** A minimal AURA integration tracks three fields: start time, end time, and pass/fail. Full conformance scoring is opt-in. Integrations can adopt AURA incrementally.

## 11. Versioning

This specification uses [semantic versioning](https://semver.org/):

- **Major** version bump: Breaking changes to the data model or metric definitions.
- **Minor** version bump: New optional fields, new metrics, or new failure types.
- **Patch** version bump: Clarifications, typo fixes, and additional examples.

The `schema_version` field in the metrics output must match the major.minor version of the schemas used. Consumers should accept any patch version within their supported major.minor range.

Current version: **0.1.0**
