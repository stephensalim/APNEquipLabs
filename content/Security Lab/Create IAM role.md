---
title: "Create IAM Role"
weight: 2
draft: true
---

1. Open the IAM console at https://console.aws.amazon.com/iam/

2. In the navigation pane, choose Roles.

3. Click on **Create role**.

4. Select **AWS Service** for the trusted entity, and Choose **EC2** as the service that will use this role.

	![](/Security Lab/images/image24.png) 

5. Click **Next permission** 

6. Under Attach permission policies, seach for **CloudWatchAgentServerPolicy** and select it.

7. Click **Next:Tags**

8. Click **Next:Reviews**

9. Type in your Role name, and click **Create role**
