---
title: "State 2: Check Precedent"
weight: 3
chapter: false
pre: " <b> 2.3.3. </b> "
---

# 2.3.3. State 2 – Check Precedent (Lambda `lambda_history`)

**Function:** Query the DynamoDB `incident_history` table to determine whether this attack type has occurred on the resource within the last 90 days.

**Query scheme:**
- **Partition Key:** `finding_type`
- **Sort Key:** `target_id`
- **GSI:** `timestamp` (filter by time, TTL 90 days)

**Output:**

| Field | Meaning |
|-------|---------|
| `has_precedent` | `true`/`false` — whether a precedent exists |
| `previous_action` | Action taken previously |
| `recurrence_count` | Number of occurrences in 90 days |

This result is added to `historical_context` and passed to the Bedrock Agent to help identify recurring attack patterns.
