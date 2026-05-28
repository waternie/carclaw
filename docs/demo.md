# CarClaw Demo

This demo shows the intended public workflow for an evidence-first vehicle diagnostic agent.

## Demo Goal

Turn a vehicle symptom or DTC into a diagnostic report that separates:

- user statement
- diagnostic tool result
- service-manual evidence
- AI inference
- missing evidence
- next inspection steps

## Demo Flow

```text
User symptom / DTC
  -> diagnostic_read_dtc
  -> knowledge_query_dtc_solution
  -> evidence-first reasoning
  -> auditable diagnostic report
```

## Example User Input

```text
发动机故障灯亮了，诊断仪报 P0300，帮我判断下一步该查什么。
```

## Expected Agent Behavior

1. Normalize the DTC: `P0300`.
2. Query evidence with `knowledge_query_dtc_solution`.
3. Summarize only evidence-backed findings.
4. List likely causes with clear labels:
   - manual evidence
   - tool result
   - AI inference
5. Provide next inspection steps.
6. State missing evidence instead of inventing details.

## Example Output Shape

```markdown
# Vehicle Diagnostic Report

## Summary

- User symptom:
- Vehicle context:
- DTC:
- Evidence status:

## Findings

### DTC P0300

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

## What This Demo Is Not

- It is not a real vehicle control workflow.
- It does not clear DTCs.
- It does not write ECU configuration.
- It does not run actuator tests or flashing.
- It does not replace a qualified technician.

## Public Sharing Message

> CarClaw is an evidence-first workflow protocol for vehicle diagnostic AI agents. It keeps DTCs, service-manual evidence, AI inference, and missing evidence separate.
