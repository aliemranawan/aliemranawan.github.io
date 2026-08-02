# OIC Integration Design Reference

A REST-vs-SOAP decision framework and sanitized integration flow patterns for Oracle Integration Cloud (OIC), built from designing integrations between Fusion Cloud, EBS, and third-party systems.

No real endpoints, credentials, connection details, or client-specific payloads are included anywhere in this repo — everything here is a pattern, not a working connector.

## Why this exists

Most write-ups on OIC integration jump straight to "how to configure a connection" without explaining the design decision that comes first: **should this be REST or SOAP, synchronous or asynchronous, and where does the transformation logic actually belong?** That decision is where integrations succeed or fail long before configuration starts. This repo documents that decision layer.

## Structure

```
oic-integration-reference/
├── decision-framework/
│   └── rest-vs-soap-selection.md         -- criteria for choosing protocol
├── flow-patterns/
│   ├── sync-request-response.md          -- pattern + sequence diagram
│   ├── async-event-driven.md             -- pattern + sequence diagram
│   └── batch-scheduled-orchestration.md  -- pattern + sequence diagram
├── payload-mapping/
│   └── sample_field_mapping.md           -- generalized source->target mapping structure
├── error-handling/
│   └── fault-handling-patterns.md        -- retry, dead-letter, and alert patterns in OIC
└── README.md
```

## Decision framework (summary)

| Factor | Favors REST | Favors SOAP |
|---|---|---|
| Target system's native API | Modern SaaS/cloud endpoints | Legacy EBS/on-prem services often expose SOAP-only WSDLs |
| Payload structure | Simpler JSON payloads, fewer nested types | Complex nested/typed XML with strict schema (XSD) contracts |
| Latency requirement | Lower overhead, better fit for near-real-time | Higher overhead, but sometimes the only option available |
| Existing contract | New integration, no legacy constraint | Existing SOAP service already in production, changing it isn't worth the risk |

The short version: protocol choice in practice is usually dictated by what the target system actually exposes, not by a preference. The framework in `decision-framework/rest-vs-soap-selection.md` covers the less obvious cases — like when a system exposes both, and how payload complexity and error-handling needs tip the decision.

## Flow patterns

Each pattern includes a sequence diagram and a description of when to use it:

- **Sync request-response** — used when the caller needs an immediate result (e.g. real-time credit check before order submission).
- **Async event-driven** — used when the source system shouldn't block on downstream processing (e.g. HR event triggering multiple downstream system updates).
- **Batch/scheduled orchestration** — used for high-volume, non-time-sensitive data movement (e.g. nightly BICC extract feeding a downstream reporting system).

## Error handling

`fault-handling-patterns.md` covers retry strategy design (exponential backoff vs fixed interval, and when each is appropriate), dead-letter queue patterns for integrations that can't lose data, and alerting thresholds that avoid both alert fatigue and silent failures.

## How to use this repo

1. Start with the decision framework before designing a new integration — protocol choice made early is much cheaper to change than protocol choice made after building the flow.
2. Match your scenario to a flow pattern and adapt the sequence diagram to your actual systems.
3. Use the fault-handling patterns as a baseline for error handling in the integration, not an afterthought bolted on at the end.

## Tech

Oracle Integration Cloud (OIC), REST and SOAP web services. Diagrams in Mermaid.js sequence-diagram syntax (renderable directly on GitHub).

## Disclaimer

No real integration endpoints, credentials, connection configurations, or client-specific payload structures are included. All examples are generalized design patterns.
