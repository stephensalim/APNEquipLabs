---
title: '1. Configure Network'
weight: 1
draft: false
---

Before launching a new instance, you will need to configure a new VPC,
subnet, internet gateway and routes. All activities should be performed
in the **us-west-1** region (N. California).

1.  In AWS console ensure that you are accessing the West-1 region,
    navigate to VPC.

2.  In the left navigation pane click "Your VPCs"

3.  Click **Create VPC** then configure:

-   Name tag: West VPC

-   IPv4 CIDR block: 10.0.0.0/16

-   Click Yes,Create

    ![](/EC2 Lab Windows/images/image1.png)

    Next you will define and configure a public subnet in the "West VPC"
    you created:

1.  From the VPC Dashboard, navigate to the left pane and click
    **Subnets**

2.  Click the **Create Subnet** button then enter the following
    configuration:

    -   Name tag: West Public

    -   VPC: West VPC

    -   Availability Zone: Select the first AZ in the list

    -   IPv4 CIDR Block: 10.0.1.0/24

    -   Click Create

        ![](/EC2 Lab Windows/images/image2.png)

3.  Back in the VPC Dashboard in the Subnet view: select "West Public".

4.  In the **Actions** menu, select **Modify auto-assign IP Settings**

5.  Select **Enable auto-assign public IPv4 addresses** -- This setting
    provides a public iPV4 address for all instances launched into the
    "West Public" Subnet.

The "West Public" subnet is not publicly accessible until you associate
it with an Internet Gateway. The Internet Gateway is a VPC component
that allows communication between instances in your VPC and the
internet. An Internet Gateway serves two purposes: to provide a target
in your VPC route tables for internet-routable traffic, and to perform
network address translation (NAT) for instances that have been assigned
public IPv4 addresses.

In the next step you will create and assign an Internet Gateway to allow
remote connectivity to your EC2 instance.

1.  Navigate to the left side of the VPC dashboard and click: **Internet
    Gateways**

2.  Click **Create Internet Gateway** then configure the following
    settings:

    -   Name Tag: West IG

    -   Click Create

    -   Click Close

3.  Select the Internet Gateway you just created "West IG", in the
    **Actions** menu select **Attach to VPC**

    -   VPC: West VPC

    -   Click **Attach**

        You now have an Internet Gateway attached to your VPC. Now you
        need to enable traffic flow between your public subnet and the
        internet.

        Next you will create a route table for internet-bound traffic
        and add a route to the table to direct internet bound traffic to
        your internet gateway and finally associate your public subnet
        with the route table.

1.  Navigate to the VPC Dashboard, on the left side click **Route
    Tables**

-   You will see a route table for the default VPC and also one for the
    "West VPC" you created earlier, these facilitate local VPC traffic

2.  Click the **Create Route Table** button then configure:

    -   Name Tag: West Public Route Table

    -   VPC: West VPC

    -   Click **Yes,Create** button

3.  Select "West Public Route Table" and click the **Routes** tab in the
    bottom half of the page.

4.  Click **Edit** button then **Add another Route** button and
    configure:

    -   Destination: 0.0.0.0/0

    -   Target: West IG

    -   Click **Save** -- This route will direct non local traffic to
        the Internet Gateway

5.  Click the **Subnet Associations** Tab

-   Click **Edit**

-   Click **Save**

-   **Select** "West Public" Subnet -- the subnet is now public