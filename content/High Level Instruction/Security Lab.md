---
title: "Security Lab"
date: 2019-08-14T13:29:54+10:00
draft: true
---

**Amazon CloudWatch** is a monitoring and observability service built for DevOps engineers, developers, site reliability engineers (SREs), and IT managers. CloudWatch provides you with data and actionable insights to monitor your applications, respond to system-wide performance changes, optimize resource utilization, and get a unified view of operational health. CloudWatch collects monitoring and operational data in the form of logs, metrics, and events, providing you with a unified view of AWS resources, applications, and services that run on AWS and on-premises servers. You can use CloudWatch to detect anomalous behavior in your environments, set alarms, visualize logs and metrics side by side, take automated actions, troubleshoot issues, and discover insights to keep your applications
running smoothly.

In this lab we will be building an EC2 instance to emit the Operating system security log to CloudWatch Logs, and set up a CloudWatch alarm to trigger notification when a failed SSH attempt is made to the Instance. You will also learn how filter through the the security log to identiy details about the attempt.


**Challenge**

1.  Create Simple Notification Service Topic.
2.  Subscribe your email to the topic.
3. 	Create a new EC2 instance, in a public facing subnet.
4. 	Use Amazon Linux AMI 2018.03.0 Amazon Machine Image.
5. 	Use t2.micro EC2 instance size.
6.	Bootstrap the EC2 instance using the following bash script
    (Replace the <REPLACE WITH YOUR REGION> section with your region name. e.g: ap-southeast-2)

		```
        #!/bin/sh
        yum -y update
        yum install -y awslogs
        sed -i -e 's/region = us-east-1/region = <REPLACE WITH YOUR REGION>/g' /etc/awslogs/awscli.conf
        sed -i -e 's/messages/secure/g' /etc/awslogs/awslogs.conf
        sudo service awslogs start
		```

7.  Open Security Group on EC2 instance to accept SSH traffic from your IP.
8.  Confirm that the EC2 instance has pushed the /var/log/secure logs into CloudWatch Logs.
9.  Create a CloudWatch metric filter, to filter through using this pattern 
    `[Mon, day, timestamp, ip, id, status = Invalid, ...]`.
10. Create alarm on that filter to detect matched record in the log.
11. Set up the alarm to trigger the SNS notification the Sum of the event is more than 2 in 10 seconds interval.
12. Use CloudWatch Insight to identify the the IP address and the details of the invalid attempt to the EC2 instance.
 

**Documentation References:**

* [https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/MonitoringLogData.html](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/MonitoringLogData.html)
* [https://docs.aws.amazon.com/quickstarts/latest/vmlaunch/step-1-launch-instance.html](https://docs.aws.amazon.com/quickstarts/latest/vmlaunch/step-1-launch-instance.html)
* [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-cloudwatch-createalarm.html](hhttps://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-cloudwatch-createalarm.html)
* [https://docs.aws.amazon.com/sns/latest/dg/sns-tutorial-create-topic.html](https://docs.aws.amazon.com/sns/latest/dg/sns-tutorial-create-topic.html)
* [https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html)