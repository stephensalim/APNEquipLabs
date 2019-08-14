---
title: "Monitoring Lab"
date: 2019-08-13T22:10:21+10:00
draft: false
---

**Amazon CloudWatch** is a monitoring and observability service built for DevOps engineers, developers, site reliability engineers (SREs), and IT managers. CloudWatch provides you with data and actionable insights to monitor your applications, respond to system-wide performance changes, optimize resource utilization, and get a unified view of operational health. CloudWatch collects monitoring and operational data in the form of logs, metrics, and events, providing you with a unified view of AWS resources, applications, and services that run on AWS and on-premises servers. You can use CloudWatch to detect anomalous behavior in your environments, set alarms, visualize logs and metrics side by side, take automated actions, troubleshoot issues, and discover insights to keep your applications
running smoothly.

**AWS Config** is a service that enables you to assess, audit, and evaluate the configurations of your AWS resources. Config continuously monitors and records your AWS resource configurations and allows you to automate the evaluation of recorded configurations against desired configurations. With Config, you can review changes in configurations and relationships between AWS resources, dive into detailed resource configuration histories, and determine your overall compliance against the configurations specified in your internal guidelines. This enables you to simplify compliance auditing, security analysis, change management, and operational troubleshooting.

In this lab we will be building an EC2 instance and set up a CloudWatch alarm to trigger notification when the CPU is detected to be highly utilized. We will also be pushing the Operating System Systems log to cloudWatch logs. 

Plus, as a bonus you will also learn how to configure AWS Config to detect when EC2 instance is not tagged, as well as a billing alarm to keep your AWS cost in check.
