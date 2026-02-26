# AURA

**DORA for pipelines. AURA for agents.**

AURA measures the reliability and quality of AI agent output through five metrics.

[aura-metrics.github.io](https://aura-metrics.github.io/aura-metrics-specification/) · [Read the spec](spec/aura-spec.md) · [JSON Schemas](schemas/latest/)

## The Five Metrics

| Metric | Measures | Elite |
|--------|----------|-------|
| Feature Throughput | Deliverables accepted / time | ≥3/day |
| Resolution Latency | Spec received → accepted | <1 hour |
| Deliverable Failure Rate | % failing conformance | <5% |
| Recovery Efficiency | Rework overhead | <5% |
| Spec Conformance | Quality score 0–1 | ≥0.95 |

## Quick Start

Validate your AURA output against the JSON schemas:

```json
{
  "schema_version": "0.1.0",
  "change_id": "add-dark-mode",
  "started_at": "2026-02-26T10:00:00Z",
  "completed_at": "2026-02-26T10:45:00Z",
  "status": "completed",
  "metrics": {
    "resolution_latency_seconds": 2700,
    "conformance": { "overall": 0.97 }
  }
}
```

See the full [metrics output example](examples/metrics-output.json) for all fields.

## Spec Frameworks

AURA is spec-framework agnostic. Works with OpenSpec, spec-kit, Jira, Linear, Notion, plain markdown, or custom formats. The deliverable just needs acceptance criteria to score against.

## Schema Validation

```bash
# Node
npx ajv validate -s schemas/latest/metrics-output.schema.json -d my-output.json

# Python
python -m jsonschema -i my-output.json schemas/latest/metrics-output.schema.json
```

## Built on OpenTelemetry

AURA extends OTel GenAI semantic conventions with the `aura.*` namespace. See [otel/](otel/) for semantic conventions and integration guidance.

## Origin

AURA grew out of work on AgentEx — improving developer experience for AI agents to improve developer experience for humans. Created by [Eamonn Faherty](https://www.linkedin.com/in/eamonnfaherty/). Read more: [AgentEx article](https://medium.com/@eamonn.faherty_58176/agentex-the-problems-we-solved-for-devex-are-worth-more-for-agents-d7273094dad1).

## License

Apache 2.0
