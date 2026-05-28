# CarClaw Architecture

CarClaw is a protocol package, not a monolithic diagnostic product.

## Public Protocol Layer

```text
User / technician
  -> diagnostic agent
  -> diagnostic skill
  -> read-only diagnostic tool
  -> evidence lookup tool
  -> auditable report
```

## Logical Tool Contracts

| Tool | Purpose |
|---|---|
| `diagnostic_connect` | Connect to a mock or authorized diagnostic source |
| `diagnostic_read_dtc` | Read DTCs from the diagnostic source |
| `diagnostic_read_version` | Read module identity or version information |
| `knowledge_query_dtc_solution` | Query evidence and repair clues for one DTC |
| `knowledge_search` | Search service-manual evidence by symptom, part, or system |
| `case_record_event` | Optionally record diagnostic case audit events |

These contracts can be mapped to MCP tools, function calling, HTTP APIs, local scripts, or a custom agent runtime.

## Evidence Boundary

CarClaw separates:

- user statements
- diagnostic tool results
- service-manual evidence
- structured corpus records
- AI inference
- human review

Generated summaries are not the source of truth. High-risk claims must be traceable back to service-manual evidence or verified diagnostic data.

## Restricted Operations

The public community workflow is read-only by default.

It does not authorize:

- clearing DTCs
- writing ECU configuration
- custom UDS commands
- actuator tests
- calibration
- module reset or relearning
- flashing
- anti-theft or security access operations

Production systems need separate safety review, authorization, hardware validation, and compliance checks.

## Open Core Boundary

Open source:

- protocol
- read-only skill
- mock tools
- demo data
- report structure

Private or commercial:

- customer manuals
- data cleaning pipelines
- brand/model-specific parsing rules
- real hardware adapters
- production audit data
- customer deployment and support
