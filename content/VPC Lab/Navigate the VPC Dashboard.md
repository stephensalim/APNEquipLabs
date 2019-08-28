---
title: "Navigate the VPC Dashboard"
weight: 1
draft: false
---

To get started, let's take a look at the VPC Dashboard.

![](/Detailed Instructions/VPC Lab/images/image2.png)

In every region, a **default VPC** has already been created for you. So,
even if you haven't created anything in your account yet, you will see
some VPC resources already there.

In this lab, we will be using the **VPC Wizard** to create a VPC with
private and public subnets pre-configured. Once you're more familiar
with AWS networking, you can create VPCs and subnets without the Wizard
to create custom networking configurations.

During the VPC wizard set up, you will need to specify an **Elastic IP
Address (EIP)**. An Elastic IP is a static public IPv4 address that you
can attach to AWS resources, such as EC2 instances and NAT Gateways.
Elastic IPs are required for NAT Gateways.

To create one, go to **Elastic IPs** in the sidebar, and press
**Allocate new address**, **Allocate**, **Close.**

![](/Detailed Instructions/VPC Lab/images/image4.png)

Now click on **VPC Dashboard** in the top left corner to go back to the
main VPC page.

Click on **Launch VPC Wizard** to start the VPC Wizard and select '**VPC
with Public and Private Subnets'**.

![](/Detailed Instructions/VPC Lab/images/image5.png)

This option will create a VPC with a /16 CIDR block and two subnets with
/24 CIDR blocks which have 256 total IP addresses each. In each subnet,
**AWS reserves 5 IP addresses**. In this case, that leaves you 251 IP
addresses per subnet. Fill in the **VPC name** (I called my VPC
"**test"**) and select your **Elastic IP Allocation ID** from the
drop-down. For this lab, we will leave the rest of the default
configuration as is.

![](/Detailed Instructions/VPC Lab/images/image6.png)

This Elastic IP will be used to create a **Network Address Translation
(NAT) gateway** for the private subnet. NAT gateway is a managed, highly
scalable NAT service that gives your resources access to the internet
but doesn't allow anyone on the internet access to your resources. NAT
is helpful for when a resource needs to pull down updates from the
internet but should not be publicly accessible.

Click on **Create VPC**. This step will take a couple minutes. Once your
VPC has been created, click **OK**.

Wow! It only took you a few minutes to set up an entire virtual private
network, including subnets for public and private resources, routing
rules, and a scalable NAT service.

