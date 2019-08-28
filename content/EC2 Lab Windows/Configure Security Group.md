---
title: '2. Configure a Security Group'
weight: 2
draft: false
---

You will now create a security group which will allow remote desktop
(RDP) access to an EC2 instance.

A *security group* acts as a virtual firewall for your instance to
control inbound and outbound traffic. When you launch an instance in a
VPC, you can assign up to five security groups to the instance. Security
groups act at the instance level, not the subnet level. Therefore, each
instance in a subnet in your VPC could be assigned to a different set of
security groups. If you don\'t specify a particular group at launch
time, the instance is automatically assigned to the default security
group for the VPC.

1.  From the VPC dashboard, navigate left and under Security click
    **Security Groups**

2.  Click **Create Security Group** then configure:

-   Name Tag: RDP Access

-   Group Name: RDP Access

-   Description: Remote Desktop Access Group

-   VPC: West VPC

-   Click **Yes, Create**

3.  Select **RDP Access** Security Group

4.  Click the **Inbound Rules** Tab

5.  Click **Edit** then configure:

-   Type : RDP (3389)

-   Source: 0.0.0.0/0

-   Click Save

    (Note: This rule will open RDP port access to the entire internet,
    this is not recommended in a production environment. It is
    recommended that you scope this down to your local IP or subnet
    range if you plan to keep this configuration running after you
    complete the lab.

You have now completed the network configuration. You now have a VPC in
the us-west-1 region with a public subnet routed to an internet gateway.
You have also created a security group which will allow RDP access from
the internet to its members.
