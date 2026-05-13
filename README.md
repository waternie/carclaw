# CarClaw Community Protocol

CarClaw Community Protocol is an open workflow specification for building AI agents for vehicle diagnostics.

It focuses on one vertical task:

```text
Vehicle symptom or DTC
  -> evidence lookup
  -> diagnostic reasoning
  -> next inspection steps
  -> repair note
  -> auditable report
```

The goal is not to publish another generic prompt. The goal is to describe a safe, evidence-first workflow that AI agents can follow when handling vehicle fault codes, service-manual evidence, and diagnostic reports.

## What is included

This public package currently contains:

- [`AGENTS.md`](./AGENTS.md): project-level operating protocol for vehicle diagnostic agents.
- [`SKILL.md`](./SKILL.md): a community read-only diagnostic skill that can be adapted to agent runtimes.
- [`README.md`](./README.md): overview and usage notes.

The public version is intentionally small. It is meant to be copied, adapted, reviewed, and used as the starting point for a safe automotive diagnostic agent workflow.

## What is not included

This repository does not include:

- proprietary service manuals
- customer repair data
- vehicle-specific ECU configuration
- CAN ID or response ID databases
- real hardware adapters
- commercial parsing rules
- production deployment code
- credentials or private environment files

Those belong in private deployments or customer-specific implementations.

## Community scope

The open version is designed for:

- mock diagnostic environments
- read-only DTC workflows
- service-manual evidence retrieval
- agent instruction design
- safety review
- proof-of-concept demos

It should not be used to control a real vehicle without a dedicated safety review, explicit user authorization, and hardware-specific validation.

## Recommended architecture

```text
User or technician
  -> Diagnostic Agent
  -> Diagnostic Skill
  -> Read-only diagnostic tool
  -> Knowledge lookup tool
  -> Evidence-first report
```

Recommended tool categories:

- `diagnostic_connect`: connect to a mock or authorized diagnostic data source.
- `diagnostic_read_dtc`: read DTCs from the authorized source.
- `knowledge_query_dtc_solution`: query evidence for one DTC.
- `knowledge_search`: search service-manual evidence by symptom, part, or system.
- `case_record_event`: optionally record audit events for a diagnostic case.

Tool names can be mapped to your own MCP tools, function tools, API routes, or local scripts.

## Safety principles

- Treat service manuals and verified diagnostic data as the source of truth.
- Separate user statements, tool results, manual evidence, and AI inference.
- Do not invent fault definitions, torque values, fluid specs, pinouts, voltage ranges, or repair steps.
- If evidence is missing, say that the knowledge base is insufficient.
- Keep write operations out of the community skill by default.
- Never clear DTCs, write ECU configuration, run actuator tests, calibrate modules, or send custom UDS commands without explicit authorization and a separate safety workflow.

## Open source strategy

This repository is the public protocol layer.

Recommended open core split:

- Open source: protocol, read-only skill, mock tools, demo data, report structure.
- Commercial or private: customer manuals, data cleaning, brand-specific parsing, real hardware adapters, team workflow, deployment, maintenance, and support.

## Status

This is an early public protocol package. It is useful as a reference for agent workflow design, but it is not a complete diagnostic product.

