---
title: "Lớp phát hiện (Detection Layer)"
weight: 2
chapter: false
pre: " <b> 2.2. </b> "
---

# 2.2. Lớp phát hiện – Amazon GuardDuty

## Tổng quan

Lớp phát hiện có nhiệm vụ phát sinh cảnh báo khi có hành vi bất thường trên hạ tầng AWS. Trong CloudSentinel, nguồn đầu vào là **Amazon GuardDuty** – dịch vụ phát hiện đe dọa dựa trên machine learning. GuardDuty phân tích VPC Flow Logs, DNS logs, CloudTrail events để tạo các **finding** (sự cố bảo mật).

**Mục tiêu của lớp này:**
- Cung cấp cấu trúc finding chuẩn để các thành phần sau (Parser, History, Knowledge, Agent) có thể xử lý.
- Kích hoạt luồng điều phối của Step Function.

Các nội dung chi tiết:
- [GuardDuty detector](./01-guardduty-setup/01-guardduty-setup.vi.md)
- [Cấu trúc finding sau parser](./02-finding-structure/02-finding-structure.vi.md)
- [Cơ chế kích hoạt Step Function](./03-trigger-mechanism/03-trigger-mechanism.vi.md)
- [Lab: Gửi mock event](./04-lab-mock-event/04-lab-mock-event.vi.md)
- [Kiểm tra và xác minh](./05-verification/05-verification.vi.md)
- [Tổng kết lớp phát hiện](./06-summary/06-summary.vi.md)
