---
title: "IAM Lab"
weight: 3
draft: true
---

**Challange**

1. Create an IAM Group “Power_Users”. 
	Attach policy called “PowerUserAccess” to the Group
	(Review JSON content and ask Questions if in doubt)
2. Create an IAM User with console & programmatic	access (tip: make note of the programmatic info)
3. Make new user part of “power_User” group created
4. Test new user log in to AWS via console and verify access level
5. Grand “IAMFullAccess” policy to user created.
6. Verify new access level.

**Additional challange**

1. Create an IAM Role that allows an EC2 instance to read files in S3
2. Tips:
	* Trusted identity should be AWS service “EC2”
	* Use AWS Managed policy with S3 Read only access
	* Verify attributes, can you change duration to 3 hours?
	* Review JSON code, ask if in doubt

**Documentation References:**

* [https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#create-iam-users](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#create-iam-users)
* [https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups_create.html](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups_create.html)
