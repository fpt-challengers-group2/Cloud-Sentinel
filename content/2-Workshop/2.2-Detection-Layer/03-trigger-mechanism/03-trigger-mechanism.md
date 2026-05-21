---
title: "Step Function trigger mechanism"
weight: 3
chapter: false
pre: " <b> 2.2.3. </b> "
---

# 2.2.3. Step Function trigger mechanism

In the architecture design, GuardDuty sends findings to EventBridge, and an EventBridge rule triggers the Step Function. However, **in the current deployment the EventBridge rule is not fully configured**. To enable testing and operation, we use a **mock event** — sending the finding JSON directly to the Step Function via AWS CLI (see Lab instructions).
