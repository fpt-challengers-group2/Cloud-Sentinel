---
title: "Cơ chế kích hoạt Step Function"
weight: 3
chapter: false
pre: " <b> 2.2.3. </b> "
---

# 2.2.3. Cơ chế kích hoạt Step Function

Trong thiết kế kiến trúc, GuardDuty sẽ gửi finding đến EventBridge, và EventBridge rule sẽ trigger Step Function. Tuy nhiên, **trong bản triển khai hiện tại, EventBridge rule chưa được hoàn thiện**. Để vẫn có thể vận hành và thử nghiệm pipeline, chúng ta sử dụng **mock event** – gửi trực tiếp finding JSON vào Step Function bằng AWS CLI (hướng dẫn ở phần Lab).