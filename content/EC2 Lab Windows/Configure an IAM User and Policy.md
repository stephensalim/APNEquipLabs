---
title: '3. Configure an IAM User and Policy'
weight: 3
draft: false
---

Next you will work in the Identity and Access Management service (IAM)
to create a user and define a custom policy which will restrict EC2
instance launch to a specific (US-West-1) region.

1.  Navigate to the IAM console

2.  In the IAM console click **Policies** on the left

3.  Click **Create Policy**

4.  **In the Create Policy** page, click the **JSON** tab.

    ![](/EC2 Lab Windows/images/image3.png)

5.  Copy the following following text and replace all of the text in the
    JSON tab:

```
    {
    "Version": "2012-10-17",
    "Statement": [
        {
        "Action": "ec2:*",
        "Resource": "*",
        "Effect": "Allow",
                "Condition": {
                        "StringEquals": {
                        "ec2:Region": "us-west-1"
                        }
                }
        }
    ]
    }
```

6.  The JSON tab should look like this now:

    ![](/EC2 Lab Windows/images/image4.png)

    This policy will restrict EC2 Instance launch to the us-west-1 (N.
    California) region to whichever user it is assigned to.

7.  Click **Review Policy**

8.  Name the Policy: EC2\_West\_Only

-   Review the Policy, it is safe to ignore the "This policy defines
    some actions, resources, or conditions that do not provide
    permissions. To grant access, policies must have an action that has
    an applicable resource or condition" notification.

-   Click **Create Policy**

9.  Navigate to the left and click on **Users**

10. Click **Add User**

11. In the Set User Details Page configure:

-   Username: West_Admin

-   Access Type: AWS Management Console Access

-   Console Password Click **Custom** and enter a complex password

-   Uncheck require password reset

-   Click **Next: Permissions**

12. In the **Set Permissions** page select **Attach Existing Policies
    Directly**

13. In the **Filter Policies** search field enter: EC2_West_Only and
    select the result

    ![](/EC2 Lab Windows/images/image5.png)
14. Click **Review**

15. Click **Create User** then **Close**
