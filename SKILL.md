---
name: vehicle-diagnosis-community
description: Read-only vehicle diagnostic workflow. Use when the user asks about vehicle symptoms, fault lights, DTCs, diagnostic checks, or repair guidance. This community skill retrieves DTCs and service-manual evidence, then produces an evidence-first report.
---

# Vehicle Diagnosis Community Skill

This is the public read-only diagnostic skill for CarClaw-style agents.

It is designed for mock environments, authorized read-only diagnostic data, and service-manual evidence lookup. It does not authorize clearing codes, writing ECU configuration, running actuator tests, calibration, flashing, or custom UDS commands.

## Trigger Examples

Use this skill when the user says things like:

- "Check this vehicle fault"
- "The engine light is on"
- "Read the fault codes"
- "What does P0300 mean?"
- "Diagnose this DTC"
- "Search the manual for this symptom"
- "帮我诊断一下车辆故障"
- "发动机故障灯亮了"
- "读一下故障码"

## Required Tool Capabilities

Map these logical tools to your runtime, MCP server, function tools, or local APIs.

### `diagnostic_connect`

Connect to a mock or authorized diagnostic data source.

Suggested input:

```json
{
  "target": "mock-or-authorized-source"
}
```

Expected output:

```json
{
  "connected": true,
  "source": "mock",
  "metadata": {}
}
```

### `diagnostic_read_dtc`

Read diagnostic trouble codes from an authorized source.

Suggested input:

```json
{
  "ecu": "optional",
  "profile": "optional"
}
```

Expected output:

```json
{
  "dtcs": ["P0300"],
  "raw": {},
  "source": "diagnostic_tool"
}
```

### `knowledge_query_dtc_solution`

Query service-manual evidence for one DTC.

Suggested input:

```json
{
  "dtc": "P0300",
  "include_evidence": true,
  "include_diagnostic_checks": true
}
```

Expected output:

```json
{
  "dtc": "P0300",
  "description": "",
  "evidence": [],
  "possible_causes": [],
  "diagnostic_checks": [],
  "repair_actions": [],
  "confidence": 0.0,
  "review_status": "unreviewed"
}
```

### `knowledge_search`

Search evidence by symptom, component, system, or keyword when no DTC is available.

Suggested input:

```json
{
  "query": "rough idle",
  "type": "all"
}
```

## Workflow

### Step 1: Classify the request

Identify whether the user provided:

- a DTC
- symptoms
- vehicle context
- diagnostic tool output
- a request to read DTCs
- a repair or inspection question

### Step 2: Retrieve evidence

If DTCs are already present:

1. Normalize the DTC codes.
2. Call `knowledge_query_dtc_solution` for each code.
3. Keep evidence references for the final report.

If the user asks to read DTCs:

1. Call `diagnostic_connect`.
2. If connected, call `diagnostic_read_dtc`.
3. For each DTC, call `knowledge_query_dtc_solution`.

If no DTC is available:

1. Call `knowledge_search` with the symptom or system.
2. Ask for missing vehicle context only if evidence lookup cannot proceed.

### Step 3: Reason with boundaries

Separate:

- user statement
- diagnostic tool result
- service-manual evidence
- AI inference
- missing evidence

Do not turn inference into fact. Do not invent values or procedures.

### Step 4: Produce the report

Use this format:

```markdown
# Vehicle Diagnostic Report

## Summary

- User symptom:
- Vehicle context:
- DTCs:
- Evidence status:

## Findings

### DTC {{code}}

- Description:
- Evidence:
- Possible causes:
- Recommended checks:
- Repair actions:
- Confidence:
- Review status:

## Missing Information

- ...

## Next Steps

1. ...
2. ...
3. ...
```

## Safety Rules

- Read-only tools are allowed for mock or authorized sources.
- Do not clear DTCs in this community workflow.
- Do not send custom UDS commands in this community workflow.
- Do not run actuator tests, calibration, adaptation, flashing, or write operations.
- If the user requests a restricted operation, explain that it requires a separate approved safety workflow.
- If evidence is missing, say "the current knowledge base does not have enough evidence" and recommend what data is needed next.

## Severity Guidance

Use conservative severity labels:

| Severity | Meaning | Suggested response |
|---|---|---|
| Safety critical | May affect braking, steering, high voltage, airbags, fire risk, or drivability safety | Stop and escalate to qualified inspection |
| High | May damage major components or cause unsafe driving conditions | Inspect soon and avoid unnecessary driving |
| Medium | Affects performance, emissions, comfort, or reliability | Diagnose and repair in a planned workflow |
| Low | History code, intermittent issue, or insufficient evidence | Monitor, collect more data, and verify |

Do not claim a severity level is certain unless evidence supports it.

## Example

User:

```text
The engine light is on. DTC P0300 was reported.
```

Agent behavior:

1. Call `knowledge_query_dtc_solution` for `P0300`.
2. Summarize evidence.
3. List possible causes only if supported by evidence.
4. Recommend inspection steps.
5. Mark gaps and review status.

