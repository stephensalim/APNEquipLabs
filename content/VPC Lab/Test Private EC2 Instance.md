---
title: "Test Private EC2 Instance"
weight: 5
draft: false
---

Optionally, you can go through the same process in the last two sections
in order to test access to a private EC2 instance. The only difference
will be in the **Configure Instance Details** section, you will select
the **Private subnet**. Also, if you don't want to go through attaching
an Elastic IP, in the same section, you can select **Enable** under
**Auto-assign Public IP**. Remember that this is not best practice for
public facing resources, but in this case the instance will not be
reachable anyways because the private subnet does not have an IGW route.
We just want a public IP to try to access, and for this, the
automatically assigned public IP is sufficient. Additionally, you will
want to open up your security group from the beginning. That way, this
private instance will be the same in every way to the public instance
you just created except that it does not have a route to an IGW and thus
cannot be accessed publicly.
