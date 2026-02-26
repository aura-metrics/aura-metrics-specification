# AURA Integrations

Integrations connect AI agent tooling to the AURA metrics framework. An integration observes agent behavior, tracks deliverable lifecycle phases, and emits metrics in the AURA schema format.

## Planned Integrations

| Integration | Status |
|-------------|--------|
| Claude Code (via hooks) | In progress |
| Cursor | Planned |
| Windsurf | Planned |
| GitHub Actions | Planned |
| Custom (bring your own) | Documented below |

## What an Integration Does

An AURA integration is responsible for:

1. **Detecting deliverable boundaries** — Identify when a deliverable starts and ends. This may be triggered by a user command, a spec file change, or a ticket status transition.

2. **Tracking phases** — Record timestamps for phase transitions (propose → specs → design → tasks → apply → verify → archive). Not all phases are required.

3. **Counting tool calls** — Track how many tool calls the agent makes and categorize them by tool name.

4. **Detecting recovery cycles** — Identify when the agent retries after a verification failure.

5. **Scoring conformance** — Evaluate the deliverable against its spec using the four conformance dimensions (functional, correctness, constraints, iteration penalty).

6. **Emitting metrics** — Produce a JSON record conforming to [`metrics-output.schema.json`](../schemas/latest/metrics-output.schema.json) when the deliverable completes.

## Building a Custom Integration

1. Read the [AURA specification](../spec/aura-spec.md) for metric definitions and lifecycle details
2. Use the [JSON schemas](../schemas/latest/) to validate your output
3. Optionally emit [AURA events](../schemas/latest/aura-event.schema.json) for real-time streaming
4. Optionally instrument with [OpenTelemetry](../otel/) for observability integration

Minimal viable integration: track `started_at`, `completed_at`, and `status`. Everything else is optional but improves the metrics you can compute.
