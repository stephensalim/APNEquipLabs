---
title: "Monitor EC2 Security Log"
weight: 6
draft: true
---

In this section, we will explore CloudWatch to view EC2 System Logs

1. Got to CloudWatch Console, by typing in **CloudWatch** in the search bar and and press enter.

2. On the left hand side of the menu, click on **Logs**.

3. Type in **/var/log/messages** in the filter field.

	![](/Monitoring Lab/images/image25.png)

4. Click on the **/var/log/messages** log group, and locate your EC2 instance ID, then 	click on it.
	
	![](/Monitoring Lab/images/image26.png)

5. You should see all the System Log of the Operating Systems in the CloudWatch Logs.

	![](/Monitoring Lab/images/image27.png) 
 
6. These log is generated from below User Data Section of the EC2 instance. This is 	how you would push any machine log from within your operating system to CloudWatch.

	```
	....
	# THIS COMMAND WILL INSTALL AWSLOGS TOOL
	yum install -y awslogs
	sed -i -e 's/region = us-east-1/region = <REPLACE WITH YOUR REGION>/g' /etc/awslogs/awscli.conf
	sudo service awslogs start
	....
	```
