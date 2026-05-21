---
title: "Finding structure after parser"
weight: 2
chapter: false
pre: " <b> 2.2.2. </b> "
---

# 2.2.2. Finding structure (after Lambda Parser)

The system uses `lambda_parser` to extract key fields from raw GuardDuty findings. The output is a normalized JSON with the following fields:

| Field | Meaning | Example | Source in raw finding |
|-------|---------|---------|-----------------------|
| `finding_id` | Incident UUID | `"abc123-def456"` | `detail.id` |
| `finding_type` | Attack type | `"UnauthorizedAccess:EC2/SSHBruteForce"` | `detail.type` |
| `severity` | Severity (0.1 – 8.9) | `5.0` | `detail.severity` |
| `resource_type` | Resource type | `"Instance"` | `detail.resource.resourceType` |
| `target_id` | Resource ID | `"i-0123456789abcdef0"` | `detail.resource.instanceDetails.instanceId` |
| `region` | AWS region | `"ap-southeast-1"` | `detail.region` |
| `attacker_ip` | Attacker IP (if any) | `"203.0.113.45"` | `detail.service.action.networkConnectionAction.remoteIpDetails.ipAddressV4` |
| `attacker_location` | Geolocation | `"Hanoi, Vietnam"` | `detail.service.action.networkConnectionAction.remoteIpDetails.country.countryName` |
| `title` | Title | `"SSH brute force from known malicious IP"` | `detail.title` |
| `description` | Detailed description | `"The remote host ..."` | `detail.description` |
| `timestamp` | Time (ISO) | `"2026-04-26T12:34:56Z"` | `detail.createdAt` |
