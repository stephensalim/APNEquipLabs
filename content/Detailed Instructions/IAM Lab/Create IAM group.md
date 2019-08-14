---
title: "Create IAM Group"
weight: 1
draft: false
---

1.  To get started, go to the **IAM dashboard** in the AWS console.

An IAM group is a collection of users. Groups are often based on job
function or role. They allow you to manage permissions by applying
policies to each group rather than individual users.

2.  Go to **Groups** in the sidebar and click on **Create New Group.**

3.  Type in Power_Users as the Group Name. Go to the Next Step.

	Now we will attach a policy to the group. Policies are what give IAM
	entities permissions. AWS provides managed policies for many common
	access needs, but you can also write your own custom policies if needed.
	
	For this lab, we are attaching an AWS managed policy called
	**PowerUserAccess**. This policy allows all AWS actions except for most
	IAM actions. This will prevent **Power_Users** from giving themselves
	more permissions.

4.  Attach this policy to your group by typing **power** into the search
    bar, and then selecting the correct policy. Go to the **Next Step**
    and then **Create Group**.

	![](/Detailed Instructions/IAM Lab/images/image2.jpg)

	The group **Power_Users** is now created and you are back on the
	**Groups** page. Click on the **Power_Users** group and go to the
	**Permissions** tab.

	We're not going in depth on IAM policy documents in this lab, but if
	you're curious, you should view the JSON content of the
	**PowerUserAccess** policy by clicking on **Show Policy**.
	
	Awesome! You just created an IAM group with a policy attached which will
	give any user in this group permissions to do anything except for most
	IAM actions, such as giving themselves or others additional permissions.
	Next, we will create a user to put into this group.
