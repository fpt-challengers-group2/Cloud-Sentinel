---
title: "Cấu trúc finding sau parser"
weight: 2
chapter: false
pre: " <b> 2.2.2. </b> "
---

# 2.2.2. Cấu trúc một finding (sau khi qua Lambda Parser)

Hệ thống sử dụng Lambda `lambda_parser` để trích xuất các trường quan trọng từ raw GuardDuty finding. Kết quả là một JSON chuẩn có các trường sau:

| Trường | Ý nghĩa | Ví dụ | Nguồn từ raw finding |
|--------|---------|-------|----------------------|
| `finding_id` | UUID của sự cố | `"abc123-def456"` | `detail.id` |
| `finding_type` | Loại tấn công | `"UnauthorizedAccess:EC2/SSHBruteForce"` | `detail.type` |
| `severity` | Mức độ (0.1 – 8.9) | `5.0` | `detail.severity` |
| `resource_type` | Loại tài nguyên | `"Instance"` | `detail.resource.resourceType` |
| `target_id` | ID tài nguyên | `"i-0123456789abcdef0"` | `detail.resource.instanceDetails.instanceId` |
| `region` | Vùng AWS | `"ap-southeast-1"` | `detail.region` |
| `attacker_ip` | IP tấn công (nếu có) | `"203.0.113.45"` | `detail.service.action.networkConnectionAction.remoteIpDetails.ipAddressV4` |
| `attacker_location` | Vị trí địa lý | `"Hanoi, Vietnam"` | `detail.service.action.networkConnectionAction.remoteIpDetails.country.countryName` |
| `title` | Tiêu đề | `"SSH brute force from known malicious IP"` | `detail.title` |
| `description` | Mô tả chi tiết | `"The remote host ..."` | `detail.description` |
| `timestamp` | Thời gian (ISO) | `"2026-04-26T12:34:56Z"` | `detail.createdAt` |