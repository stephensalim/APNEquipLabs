---
title: "Monitoring Lab"
date: 2019-08-14T13:30:05+10:00
draft: true
---


**Challange**

1. Create Simple Notification Service
2. Launche Elastic Compute Cloud EC2.
3. Configure a CloudWatch Alarm.
4. Configure AWS Config to check if EC2 has CostCenter Tag associated.
5. Setup AWS Config Remediation action to stop EC2 instance for the check.  

**Additional challange**

6. Setup AWS Config Remediation action to stop EC2 instance for the check.  

Hint: Enable Configuration Recorder by Setting up AWS Config without a rule first, then configure rule separately after.

**Documentation References:**

* [https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#create-iam-users](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#create-iam-users)
* [https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups_create.html](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups_create.html)
* [https://docs.aws.amazon.com/config/latest/developerguide/stop-start-recorder.html](https://docs.aws.amazon.com/config/latest/developerguide/stop-start-recorder.html)
* [https://docs.aws.amazon.com/config/latest/developerguide/gs-console.html](https://docs.aws.amazon.com/config/latest/developerguide/gs-console.html)