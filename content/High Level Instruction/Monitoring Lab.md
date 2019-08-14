---
title: "Monitoring Lab"
date: 2019-08-14T13:30:05+10:00
draft: false
---

**Amazon CloudWatch** is a monitoring and observability service built for DevOps engineers, developers, site reliability engineers (SREs), and IT managers. CloudWatch provides you with data and actionable insights to monitor your applications, respond to system-wide performance changes, optimize resource utilization, and get a unified view of operational health. CloudWatch collects monitoring and operational data in the form of logs, metrics, and events, providing you with a unified view of AWS resources, applications, and services that run on AWS and on-premises servers. You can use CloudWatch to detect anomalous behavior in your environments, set alarms, visualize logs and metrics side by side, take automated actions, troubleshoot issues, and discover insights to keep your applications
running smoothly.

**AWS Config** is a service that enables you to assess, audit, and evaluate the configurations of your AWS resources. Config continuously monitors and records your AWS resource configurations and allows you to automate the evaluation of recorded configurations against desired configurations. With Config, you can review changes in configurations and relationships between AWS resources, dive into detailed resource configuration histories, and determine your overall compliance against the configurations specified in your internal guidelines. This enables you to simplify compliance auditing, security analysis, change management, and operational troubleshooting.

In this lab we will be building an EC2 instance and set up a CloudWatch alarm to trigger notification when the CPU is detected to be highly utilized. We will also be pushing the Operating System Systems log to cloudWatch logs. 

Plus, as a bonus you will also learn how to configure AWS Config to detect when EC2 instance is not tagged, as well as a billing alarm to keep your AWS cost in check.


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
        yum -y install stress
        yum install -y awslogs
        sed -i -e 's/region = us-east-1/region = <REPLACE WITH YOUR REGION>/g' /etc/awslogs/awscli.conf
        sudo service awslogs start
        stress -c 1 --backoff 300000000 -t 30m
		```

7.  Enabled detailed monitoring in EC2 instance.
8.  Create a Cloudwatch Alarm to send an SNS notification to the Topic you created above, when the CPU Utilization is above 60% after 5 minutes.
9.  Wait 5 minutes, and confirm you are receiving the Alert.
10. Confirm that the EC2 instance has pushed the /var/log/messages logs into CloudWatch Logs.
11. Configure AWS Config to check if EC2 has "CostCenter" Tag associated to it.
12. Confirm your EC2 instance is marked as non-compliant.
13. Add "CostCenter" tag into the EC2 instance and confirm in AWS Config it is now compliant.
14. Create CloudWatch Alarm to send notification if your AWS bill is estimated to go above USD 200.
 

**Documentation References:**

* [https://docs.aws.amazon.com/config/latest/developerguide/managing-aws-managed-rules.html](https://docs.aws.amazon.com/config/latest/developerguide/managing-aws-managed-rules.html)
* [https://docs.aws.amazon.com/quickstarts/latest/vmlaunch/step-1-launch-instance.html](https://docs.aws.amazon.com/quickstarts/latest/vmlaunch/step-1-launch-instance.html)
* [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-cloudwatch-createalarm.html](hhttps://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-cloudwatch-createalarm.html)
* [https://docs.aws.amazon.com/sns/latest/dg/sns-tutorial-create-topic.html](https://docs.aws.amazon.com/sns/latest/dg/sns-tutorial-create-topic.html)
* [https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html)