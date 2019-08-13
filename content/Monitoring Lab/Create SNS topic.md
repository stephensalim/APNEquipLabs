---
title: "Create SNS Topic"
weight: 1
draft: false
---

First, let's create a Simple Notification Service (SNS) Topic so that we can notify our SysAdmin.

1.  From the AWS console click **Services** > **SNS**.

2.  Under **Common Actions** click **Create Topic.**

3.  In the **Topic Name** field , type a name for your topic that
    includes your name and optionally a **Display Name** and click
    **Create Topic**.
    
    ![](/Monitoring Lab/images/image2.png)

4.  In the Topic configuration, click **Create Subscription**.

5.  In the **Protocol** drop down select **Email** and enter a working
    email address you are able to access. Utilize a non-business email
    if there may potentially be a spam filter that will block the SNS
    messages. Click **Create Subscription**.
    
    ![](/Monitoring Lab/images/image3.png)

6.  A verification email will be sent to your address with the subject
    "AWS Notification -- Subscription Confirmation". Open the email and
    click the **Confirm Subscription** link.

7.  Your subscription should now be active and **not**
    "PendingConfirmation" under the **Subscriptions** section in the SNS
    console.
