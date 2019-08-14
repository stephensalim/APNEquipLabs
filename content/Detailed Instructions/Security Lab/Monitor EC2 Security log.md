---
title: "Monitor EC2 Security Log"
weight: 6
draft: false
---

In this section, we will explore CloudWatch to view EC2 System Logs

1. Got to CloudWatch Console, by typing in **CloudWatch** in the search bar and and press enter.

2. On the left hand side of the menu, click on **Logs**.

3. Type in **/var/log/secure** in the filter field.

	![](/Detailed Instructions/Security Lab/images/image25.png)

4. Click on the **/var/log/secure** log group, and locate your EC2 instance ID, then 	click on it.
	
	![](/Detailed Instructions/Security Lab/images/image26.png)

5. You should see all the Security Log of the Operating Systems in the CloudWatch Logs.

	![](/Detailed Instructions/Security Lab/images/image27.png) 
 
