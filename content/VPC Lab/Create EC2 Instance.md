---
title: "Create Public EC2 Instance"
weight: 3
draft: true
---

In this section, you will spin up an EC2 instance in the Public subnet
of your VPC.

To get started, go to the **Services** drop down menu in the top left
corner and go to the **EC2 dashboard**.

1.  Select Launch Instance, and then in the **Quick Start** section
    select the first Amazon Linux 2 AMI and click **Select.**

    ![](/VPC Lab/images/image11.png)

2.  In the Choose Instance Type tab, select the t2.micro instance size
    and click **Next**.

3.  On this page, you decide which network and subnet this resource will
    be put into. Change the **Network** field to the VPC that you just
    created and change the **Subnet** field to the **Public subnet**.
    Leave the other default settings as is. Click **Next**.

	![](/VPC Lab/images/image12.png)

4.  For this lab, you can accept the default values in the remaining
    steps, so finish creating this instance by clicking on **Review and
    Launch.** You will see a warning that your security group is open to
    the world. You can ignore this warning and select **Launch**.

5.  In the pop-up window, select **Proceed without a key pair** from the
    drop-down and check the **acknowledgement** box. For this lab, you
    will not need to SSH into this instance. Click **Launch Instances**
    and then **View Instances**.

**Congratulations! You have just launched a virtual server in your
private network.**
