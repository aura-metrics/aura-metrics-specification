# Changelog

All notable changes to the AURA specification.

Format based on [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]

### Added
- Section 8: Supporting Metrics — nine metrics that explain why headline metrics move
- Relationship map showing how supporting metrics feed headline metrics
- `token_usage` object (input_tokens, output_tokens, total_tokens, estimated_cost_usd) to metrics output and deliverable state schemas
- `human_interventions` count to metrics output and deliverable state schemas
- `complexity` enum to metrics output schema (top-level)
- `aura.tokens.*` and `aura.human.interventions` OTel semantic attributes

## [0.1.0] - 2026-02-26

### Added
- Initial specification with five core metrics
- JSON Schemas for deliverable state, metrics output, and events
- OTel semantic conventions for `aura.*` namespace
- Performance tier benchmarks (Elite/High/Medium/Low)
- Spec conformance scoring model
- Examples for common deliverable patterns
