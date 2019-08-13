---
title: "Test Public EC2 Instance"
weight: 4
draft: false
---

In this section, you will ping the EC2 instance that you just created
and learn more about security groups along the way.

You should be on the EC2 **Instances dashboard** from the last section,
looking at all of the EC2 instances in this region. If you just finished
the last section, your EC2 instance might still be spinning up. You can
tell by looking at the **Instance State** and **Status Checks** columns.
If you see **pending** state and/or status **Initializing**, the
instance is not ready yet.

While you're waiting for your instance to be ready, select the instance
to look at the **Description** tab. At the top of the right-hand column,
there is the information that we need to access the instance -- the IP
addresses and DNS records associated with the instance. However, you can
see that this instance doesn't have a public IP or DNS yet. We will need
at least one of these to ping this instance via the internet.

![](/VPC Lab/images/image13.png)

To fix this, we are going to attach an **Elastic IP** to the EC2
instance. First, copy the **Instance ID** of this EC2 instance by
hovering your mouse to the right of the Instance ID line in the
**Description** tab, and clicking on the **Copy to clipboard** icon that
appears.

Next, click on **Elastic IPs** in the sidebar (under the **Network &
Security** section). Create a new Elastic IP as before, except this
time, **do not** click on the **Close** button. Instead, you will:
**Allocate new address, Allocate**, and then click on the **Elastic IP**
link in the **New address request succeeded** box.

![](/VPC Lab/images/image14.png)

Now you are back on the **Elastic IPs** dashboard. Go to the **Actions**
dropdown and select **Associate address**. Paste the Instance ID that
you copied previously into the **Instance** box and click on
**Associate**, then **Close**.

Now that an Elastic IP is attached to the EC2 instance, we should be
able to ping the instance over the internet. It now has a public IP
address from the Elastic IP, and it's in the public subnet which has a
route to an IGW.

In the **Description** tab, copy the Elastic IP address.

![](/VPC Lab/images/image15.png)

To ping the instance, you need to open your CLI. On Windows, open the
**Command Prompt**. On Mac, open the **Terminal**. Type **ping,** then a
**space**, then **paste the Elastic IP** from above and click **enter**.

If the instance is reachable, we expect to see lines appearing such as

```
64 bytes from 13.237.153.185: icmp_seq=0 ttl=238 time=169.294 ms
```

However, you will see request timeouts instead. You will see lines that
say something similar to

Request timed out.

*Why aren't we able to reach this instance?*

You can confirm that the instance is in the public subnet and check the
route table associated with that subnet to make sure there is a route to
the IGW.

Next, let's check our virtual firewall configurations. As you remember,
we left the NACL of the public subnet as is, which allows all traffic by
default. So, let's check the security group associated with this
instance.

In the **EC2 dashboard**, go to the **Instances** section in the
sidebar. Select the instance that you created and look at the
**Description** tab. In the left column, click on the first **Security
groups** link. It should be called something similar to
**launch-wizard-1**.

![](/VPC Lab/images/image16.png)

You are now in the **Security Groups** dashboard. Go to the **Inbound**
tab.

Remember when we were creating the EC2 instance we only specified the
AMI, instance type, and VPC subnet. We left all the other default
settings as is. One of these default settings created this security
group, which allows all inbound access on SSH port 22.

Pings use **ICMP**, so we will need to change the security group rule to
allow ICMP traffic rather than SSH traffic. Click on the **Edit**
button.

![](/VPC Lab/images/image17.png)

Click on **SSH** to open the drop-down and change it to **All** **ICMP -
IPv4**. Click **Save**.

Since security groups are stateful, you don't need to edit the outbound
rules. The security group will allow the instance to respond to the ping
since it saw the ping arrive at the instance. This change will also take
effect immediately, so we can try to ping the instance again right away.

Go back to your CLI and hit the **up arrow** and then **enter** to try
and ping the instance again.

![](/VPC Lab/images/image18.png)

**Good job! You have successfully troubleshooted why an EC2 instance was
unreachable and then accessed it over the internet.**
