---
title: '4. Create a Windows EC2 Instance'
weight: 4
draft: false
---

You have now created a new user (West_User) and restricted it from
creating EC2 instances outside of the us-west-1 region

In this task, you will create a Windows Server 2016 EC2 instance in the
**us-west-1** (N.California) region. You will perform the following
steps as the newly created "West_Admin" user.

1.  In a new browser session (Private browsing or from another maching)
    log in as **West_Admin**

2.  Ensure that you are working in the **us-west-1** region
    (N.California)

3.  Navigate to EC2 Console and click "Launch Instance"

4.  In the search field type: Windows

5.  Hit enter and scroll through the Windows AMI's, select **Windows
    2016 Base**

6.  You can now choose an instance type, at the minimum select T2.Micro.
    Click **Next: Configure Instance Details**

7.  In the **Instance Details** page configure:

-   Network: **West VPC**

-   Subnet: **West Public**

-   Click: **"Next Add Storage"**

8.  In the Add Storage leave the defaults and click **Next: Add Tags**

9.  You can add a tag if you like, otherwise click **Next: Configure
    Security Group**

10. In the Security Group page, click **Select an existing Security
    Group**

11. In the Security Group selection, check the **RDP Access** Security
    Group

12. Click **Review and Launch** -- You may see a "Improve your instances
    security" notification. This will appear if you configured your RDP
    Security group to allow all IP's, as mentioned earlier this is only
    being used for lab purposes. For a production or post lab
    environment, you should scope this rule down to a specific IP or
    range.

13. Click **Launch**

14. You may be prompted to select an existing or create a new key pair,
    this allows you to securely connect to your instance. If you already
    have a key pair generated you can re-use that, otherwise select
    **Create a new key pair**. Download the .pem file and make note of
    where you save it, you will need it to connect later.

15. Click **Launch Instance**

16. Navigate back to the EC2 Dasboard, on the left click **Instances**

17. You should see your instance being created, mouse over the **Name**
    field and click on the pencil icon and name the instance: Windows
    West

18. Once the "Status Checks" shows "2/2 checks passed" your instance
    will be ready to connect.

19. Click on you new Instance **Windows West**, click **Actions** and
    select **Get Windows Password**

20. You will be prompted for the key pair you specified/created when you
    launched the instance, browse to it and select it.

21. Click **Decrypt Password**

22. Copy the username/password information and click **Close**

23. Click on the **Windows West** instance, copy it's public IP Address

24. Launch a Windows Remote Desktop app and connect to your new
    **Windows West** instance using the public IP and credentials from
    the AWS EC2 console.

Congratulations, you have successfully launched and connected to a
Windows EC2 instance.
