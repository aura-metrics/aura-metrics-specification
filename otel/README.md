# AURA OpenTelemetry Integration

AURA extends the [OpenTelemetry GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) with the `aura.*` attribute namespace for deliverable-level metrics.

## Namespace

All AURA attributes use the `aura.*` prefix. This avoids collision with existing OTel semantic conventions.

## Semantic Conventions

See [`semantic-conventions.yaml`](semantic-conventions.yaml) for the full attribute definitions in OTel semantic convention format.

Attribute groups:
- `aura.deliverable` — Deliverable identification and status
- `aura.spec` — Specification source metadata
- `aura.phase` — Phase lifecycle tracking
- `aura.conformance` — Quality scoring dimensions
- `aura.failure` — Failure classification
- `aura.recovery` — Recovery attempt tracking
- `aura.tokens` — Token usage tracking
- `aura.human` — Human interaction tracking
- `aura.agent` — Agent identification

## Span Hierarchy

Instrumented agents should emit spans following this hierarchy:

```
aura.deliverable (root span)
├── aura.deliverable.plan
├── aura.deliverable.execute
│   ├── aura.task
│   │   ├── gen_ai.chat (OTel GenAI standard)
│   │   └── aura.tool.call
│   └── aura.recovery.attempt
├── aura.deliverable.validate
│   ├── aura.conformance.functional
│   ├── aura.conformance.correctness
│   └── aura.conformance.constraints
└── aura.deliverable.accept
```

The root `aura.deliverable` span covers the entire deliverable lifecycle. Child spans correspond to phases and sub-operations. The `gen_ai.chat` spans use standard OTel GenAI conventions and nest within the AURA task spans.

## Metric Instruments

| Instrument | Type | Unit | Description |
|------------|------|------|-------------|
| `aura.deliverables.count` | Counter | `{deliverable}` | Total deliverables processed |
| `aura.deliverables.accepted` | Counter | `{deliverable}` | Deliverables accepted |
| `aura.deliverables.failed` | Counter | `{deliverable}` | Deliverables failed |
| `aura.resolution_latency` | Histogram | `s` | Resolution latency distribution |
| `aura.conformance.score` | Histogram | `1` | Conformance score distribution |
| `aura.recovery.attempts` | Histogram | `{attempt}` | Recovery attempts per deliverable |
| `aura.tool_calls.count` | Counter | `{call}` | Total tool calls |

Labels: `aura.deliverable.type`, `aura.agent.name`, `aura.failure.type`

## Getting Started

1. Add the `aura.*` attributes from [`semantic-conventions.yaml`](semantic-conventions.yaml) to your OTel instrumentation
2. Emit spans following the hierarchy above
3. Record metric instruments for aggregate analysis
4. Export to your OTel-compatible backend (Jaeger, Honeycomb, Grafana, etc.)

For the full specification, see the [AURA spec](../spec/aura-spec.md).
