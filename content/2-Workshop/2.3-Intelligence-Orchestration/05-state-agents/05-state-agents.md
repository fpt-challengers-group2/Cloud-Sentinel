---
title: "State 4: Invoke Agents Pipeline"
weight: 5
chapter: false
pre: " <b> 2.3.5. </b> "
---

# 2.3.5. State 4 – Invoke Agents Pipeline (Lambda `lambda_advisor`)

This is the core state of the intelligence layer. Lambda `lambda_advisor` receives an `intelligence_package` (including `finding_details`, `historical_context`, `remediation_guidelines`) and invokes two Bedrock Agents sequentially.

## Supervisor Agent

| Property | Value |
|----------|-------|
| Model | `anthropic.claude-3-5-sonnet-20240620-v1:0` |
| Alias | `TSTALIASID` (DRAFT – avoid Terraform loops) |
| Action Groups | 3 groups: Parser, History, Knowledge |

The Supervisor Agent can proactively call back Lambdas (parser, history, knowledge) via action groups to gather additional information if needed. This is the Bedrock agentic loop: the Agent decides when to fetch more data before concluding.

**Supervisor Agent output** is a standardized JSON:

```json
{
  "executive_summary": "Overview of the incident",
  "target": { "id": "...", "type": "...", "region": "..." },
  "attacker": { "ip": "...", "location": "..." },
  "historical_analysis": {
    "has_precedent": true,
    "recurrence_count": 2,
    "previous_action": "block_ip"
  },
  "remediation_guidelines": ["Guideline 1", "Guideline 2", "..."]
}
```

## Advisor Agent

| Property | Value |
|----------|-------|
| Model | `anthropic.claude-3-5-sonnet-20240620-v1:0` |
| Alias | `TSTALIASID` |
| Action Groups | None (pure LLM) |

The Advisor Agent consumes the Supervisor JSON and synthesizes a **Markdown remediation plan** in 7 sections:

1. Executive Summary
2. Threat Details
3. Immediate Remediation (specific actions: `block_ip`, `isolate_instance`, `revoke_keys`)
4. Investigation Steps
5. Long-term Recommendations
6. Verification Steps
7. Rollback Plan

The Markdown report is stored in S3 and sent to admins via Telegram for approval.
