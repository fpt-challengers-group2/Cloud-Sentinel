---
title: "Step Functions Orchestrator"
weight: 1
chapter: false
pre: " <b> 2.3.1. </b> "
---

# 2.3.1. AWS Step Functions – Orchestrator

The state machine `cloud-sentinel-orchestrator` orchestrates the pipeline with **6 sequential states**:

```
Parse Finding
    ↓
Check Precedent
    ↓
Get Knowledge
    ↓
Invoke Agents Pipeline
    ↓
Send Telegram And Wait   ← waitForTaskToken (await approval)
    ↓
Execute Remediation
```

> **Design note:** The original design described 7 states (separating "Send Telegram" and "Wait for approval"). In the current implementation these are combined into a single `Send Telegram And Wait` state using Step Functions' `waitForTaskToken` — it sends the message and pauses until a callback approval webhook is received. Functionally equivalent.

Key state machine settings:

| Property | Value |
|----------|-------|
| Name | `cloud-sentinel-orchestrator` |
| Logging | `ALL` (log all transitions) |
| Region | `ap-southeast-1` |
| Type | Standard Workflow |
