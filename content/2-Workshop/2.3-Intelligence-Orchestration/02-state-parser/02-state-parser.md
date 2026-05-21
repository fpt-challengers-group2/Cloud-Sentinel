---
title: "State 1: Parse Finding"
weight: 2
chapter: false
pre: " <b> 2.3.2. </b> "
---

# 2.3.2. State 1 – Parse Finding (Lambda `lambda_parser`)

**Function:** Accept the input finding (from mock event or EventBridge) and normalize it into a unified JSON structure for downstream states.

**Input:** Raw finding JSON (GuardDuty `detail` wrapper). `lambda_parser` extracts required fields from `detail.*` (see table in 2.2.2.2).

**Output – normalized finding structure (repeated from 2.2.2.2):**

| Field | Meaning | Example |
|-------|---------|---------|
| `finding_id` | Incident UUID | `"mock-finding-001"` |
| `finding_type` | Attack type | `"Recon:EC2/Portscan"` |
| `severity` | Severity (0.1–8.9) | `5.0` |
| `resource_type` | Affected resource type | `"EC2"` |
| `target_id` | Resource ID | `"i-0123456789abcdef0"` |
| `region` | AWS region | `"ap-southeast-1"` |
| `attacker_ip` | Attacker IP (if any) | `"203.0.113.45"` |
| `attacker_location` | IP geolocation | `"Hanoi, Vietnam"` |
| `title` | Incident title | `"Port scan from malicious IP"` |
| `description` | Detailed description | `"EC2 instance was probed..."` |
| `timestamp` | Occurrence time (ISO 8601) | `"2026-04-26T10:00:00Z"` |
