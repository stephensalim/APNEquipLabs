---
title: "2. Create IAM User"
weight: 2
draft: false
---

It is best practice to create a different user for each person who needs
AWS access. A user is associated with permanent credentials and has
permissions based off of policies attached directly to the user or
policies attached to a group to which the user belongs.

1.  Go to **Users** in the sidebar and click on **Add user.**

	Enter a unique name as the **User name**. In this walkthrough, I chose
	**Emily** as the username.
	
	For this lab, we will choose to give this user both **Programmatic
	access** and **AWS Management Console access**. In practice, follow the
	principle of **least privilege** and only give users the type(s) of
	access necessary for their work. This prevents unnecessary extra
	credentials from being created.
	
	![](/IAM Lab/images/image3.png)

2. For **console access**, you can choose to have AWS **autogenerate** a
password or you can type in a **custom** password here. Either way, you
have the option of emailing this password to the user at the end of this
process. Click **Next.**

3. Now we will **Add user** to the **group** we made in the previous step,
**Power\_Users**. There are other options which would attach policies
directly to this user, but as mentioned before, using IAM groups is a
best practice in order to simplify policy management.

	If the top of your page doesn't look like the screenshot below, you need
to scroll up.

	![](/IAM Lab/images/image4.png)

4. Go to **Next: Review** and then **Create user**.

	After clicking create user, you will be presented with all of the
information you will need to get that user started working in AWS.

	![](/IAM Lab/images/image5.png)

5. For **programmatic access**, the user will need their **Access key ID**
and **Secret access key**. Take note of the Secret access key now or
**Download .csv.** *You will never be able to retrieve the secret key
once you leave this page.*

6. For **console access**, the user will need the **IAM users sign-in
link:** https://\[account ID or alias\].signin.aws.amazon.com/console.
Or, they can go to https://console.aws.amazon.com and login as long as
they know the **account ID or alias**.

7. They will also need to know their **username** and **password**. Take
note of the password, **Download .csv**, or **Send** an **email** to the
user with their login information. If you decide to go down the email
route, remember to fill in the username in the pre-written email and
send the password in a separate email. You will not be able to see the
password once you leave this page.

	For this lab, you will need to have the account ID/alias, username, and
password.

8. Once you're done, **Close** out of this page.

9. Back on the **Users** page, click on the **User name** of the user that
you just created.

	![](/IAM Lab/images/image6.png)

10. Go to the **Security credentials** tab. If you forgot to take note of
the access keys or password before, or if credentials have been
compromised, here is where you can reset the password (click on **Manage
password**), change access keys to **inactive**, and create new access
keys.

Great! You have just created an IAM user with both programmatic and
console access. You didn't have to attach policy permissions directly to
the user since you had a group created already with the correct
permissions.
