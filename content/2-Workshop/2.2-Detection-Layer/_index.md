---
title: "Detection Layer"
weight: 2
chapter: false
pre: " <b> 2.2. </b> "
---

# 2.2. Detection Layer – Amazon GuardDuty

## Overview

The Detection Layer is responsible for generating alerts when anomalous activity occurs in the AWS infrastructure. In CloudSentinel, the input source is **Amazon GuardDuty** — a threat detection service powered by machine learning. GuardDuty analyzes VPC Flow Logs, DNS logs, and CloudTrail events to produce **findings** (security incidents).

**Goals of this layer:**
- Provide a standardized finding structure so downstream components (Parser, History, Knowledge, Agent) can process it.
- Trigger the orchestration flow in the Step Function.

Detailed contents:
- [GuardDuty detector](./01-guardduty-setup/01-guardduty-setup.md)
- [Finding structure after parser](./02-finding-structure/02-finding-structure.md)
- [Step Function trigger mechanism](./03-trigger-mechanism/03-trigger-mechanism.md)
- [Lab: Send mock event](./04-lab-mock-event/04-lab-mock-event.md)
- [Verification](./05-verification/05-verification.md)
- [Detection layer summary](./06-summary/06-summary.md)
