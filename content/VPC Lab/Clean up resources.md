---
title: "6. Clean Up Resources"
weight: 6
draft: false
---

If you want to clean up your account to get rid of everything we created
during this lab, follow the instructions in this section. You can also
leave your lab environment running if you want to test other AWS
networking concepts.

First, you will need to terminate the EC2 instances that are running in
the VPC. In the **EC2 dashboard**, select the public instance (as well
as the private instance if you created one), go to the **Actions**
dropdown, go to **Instance State**, and then **Terminate**. In the
pop-up window, click on **Yes, Terminate.** You may need to wait a
minute for the instances to finish shutting down. Watch the **Instance
State** column and wait for the status to change from **shutting-down**
to **terminated**.

Now we will delete the NAT gateway that the VPC wizard created. Go to
the **Services** dropdown in the top left corner and select the **VPC**
**dashboard**. Go to the **NAT Gateways** section in the sidebar. If
there are multiple NAT Gateways, you can look at the VPC column to
confirm which one belongs to your VPC. Select that NAT gateway and go to
the **Actions** dropdown. Select **Delete NAT Gateway**. In the pop-up
window, click on **Delete NAT Gateway** again. It may take a minute for
the NAT gateway to delete. Wait for the status to change from
**deleting** to **deleted** to make sure the VPC deletion will work.

Elastic IP addresses are completely free as long a they are attached to
a resource. However, if the NAT gateway or EC2 instance they were
attached to is terminated or deleted, the unattached EIP will incur a
small monthly charge. To clean up, navigate to Elastic IPs on the
sidebar, and Release each EIP you have allocated.

Now you can finally delete your VPC. Go to the **Your VPCs** section in
the sidebar. Select your VPC, go the **Actions** dropdown, and choose
**Delete VPC**. In the pop-up window, click **Yes, Delete**. This will
take a minute or so to complete.
