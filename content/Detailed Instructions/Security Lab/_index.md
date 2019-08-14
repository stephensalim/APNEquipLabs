---
title: "Security Lab"
date: 2019-08-13T22:10:21+10:00
draft: false
---

...... [DIAGRAM]


**Amazon CloudWatch** is a monitoring and observability service built for DevOps engineers, developers, site reliability engineers (SREs), and IT managers. CloudWatch provides you with data and actionable insights to monitor your applications, respond to system-wide performance changes, optimize resource utilization, and get a unified view of operational health. CloudWatch collects monitoring and operational data in the form of logs, metrics, and events, providing you with a unified view of AWS resources, applications, and services that run on AWS and on-premises servers. You can use CloudWatch to detect anomalous behavior in your environments, set alarms, visualize logs and metrics side by side, take automated actions, troubleshoot issues, and discover insights to keep your applications
running smoothly.

In this lab we will be building an EC2 instance to emit the Operating system security log to CloudWatch Logs, and set up a CloudWatch alarm to trigger notification when a failed SSH attempt is made to the Instance. You will also learn how filter through the the security log to identiy details about the attempt.