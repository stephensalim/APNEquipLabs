---
title: "Create IAM Role"
weight: 4
draft: true
---

IAM Roles can be assumed by AWS services, IAM users, or applications.
They are assigned temporary rather than permanent credentials whenever
assumed. Using roles for privileged permissions sets (such as the
**PowerUserAccess** policy) can help improve your security posture since
credential exposure is minimized.

1.  Go to **Roles** in the sidebar and click on **Create Role.**

2.  On the **Select type of trusted identity** page, you decide who or
    what will be able to assume this role. For this lab, we will create
    a role that allows an EC2 instance to read files in S3. Therefore,
    we will stay on the **AWS service** tab and select **EC2**. Go to
    **Next: Permissions**.

	![](/IAM Lab/images/image10.png)

3.  Attach a managed policy with S3 Read Only access to the role by
    typing **s3** into the search bar, and then selecting the
    **AmazonS3ReadOnlyAccess** policy. Go to the **Next: Review**.

	![](/IAM Lab/images/image11.png)

4.  Give your role a descriptive name, such as **EC2\_S3ReadOnly** and
    edit the **role description** to be a helpful summary of what this
    role is. When you're done, **Create Role**.

5.  You are now back on the **Roles** page. Enter the name of the role
    you just created into the search bar and click on the role name.

	![](/IAM Lab/images/image12.png)

6.  You are now on the **Summary** page of the role you just created.
    Here you can view and edit attributes of the role, such as how long
    the role's temporary credentials last. The default value as you can
    see below is 1 hour but can be as long as 12 hours.

	![](/IAM Lab/images/image13.png)

**Awesome! You've just created an IAM role which will allow EC2
instances to read objects in S3.**
