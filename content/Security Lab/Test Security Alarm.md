---
title: "8. Test Security Alarm"
weight: 8
draft: false
---

In this scenario, we will test the alarm we configured previously by attempting an SSH session using a bad user cred.

1. Go back to the EC2 Console.

2. Select the EC2 instance you launched before.

3. Under the **Description** Tab take note of the Public IP address of the instance.

4. Locate the Security Group Section, and click on the Security group name you've attached to the instance.

    ![](/Security Lab/images/image35.png)

5. You should now be taken to the Security Group console as below.

6. Ensure the Security group is selected, and click on **Inbound Tab**.

7. Click on **Edit**.

8. Click on **Add rule**, then add a rule to allow SSH port open to **My IP**, then click **Save**

    ![](/Security Lab/images/image36.png)

9.  Open up your Terminal Console or Putty if you are running windows, and execute below commands multiple times consecutively.
    To emulate login attempt of false user.

    `ssh badpeople@<public ip address of your EC2 instance >`

    ![](/Security Lab/images/image37.png)

10. Go back to your CloudWatch Console.

11. Click on **Alarms > Alarm**, notice that it has gone to ALARM state and click on the Alarm.

12. Click on the Alarm that has been triggered.
    Under the History Section you should see event history of the Alarm being triggered, then it sends out a notification to the SNS topic.
    Which should result in your mailbox.

    ![](/Security Lab/images/image38.png)

Now that we have triggered the Alarm, let's query the logs and find out a bit more information about this malicious activity.