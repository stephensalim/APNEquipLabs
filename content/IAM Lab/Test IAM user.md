---
title: "3. Test IAM User"
weight: 3
draft: false
---

In this section, we will put your IAM User to the test. You will want to
stay logged in as you currently are while at the same time testing the
user's access. To do this, you will need to either open a **different
browser** or open **private browsing** in your current browser. You will
use this new window to sign in as the user you just created. If you
didn't take note of the password or login link previously, now is the
time to do so.

1. Go to your account's IAM user sign-in page. You could also just go to
console.aws.amazon.com and enter your account ID or alias. Sign in using
the IAM User **username** and **password**.

	You can tell which user you are signed in as by looking at the top right
corner of the console. The dropdown menu will now be titled **User @
Account ID/Alias.**

2. We know from the IAM Group section that this user has access to
everything except most IAM permissions. Go to the **IAM dashboard** to
see this in action.

	Now instead of seeing a summary of IAM users, groups, and roles, you
will see multiple errors.

	![](/IAM Lab/images/image7.png)

3. Now we will give the user extra permissions in order to allow access to
IAM.

	Go to your other browser. You should still be in the **Users** dashboard
on this user's page. Go to the **Permissions** tab and click on **Add
permissions**. Adding permissions this way will attach policies directly
to the user rather than to the entire group.

	![](/IAM Lab/images/image8.png)

	This time, we will **Attach existing policies directly**. Type IAM into
the search bar to find a managed policy which will give this user access
to IAM. Select the **IAMFullAccess** policy and click on **Next:
Review** and then **Add permissions**.

	![](/IAM Lab/images/image9.png)

	Go back to the browser where you are signed in as the user. IAM policies
are evaluated every time an action is attempted, so we just need to
**refresh** the page to see the changes we made take effect. The IAM
dashboard should now show content rather than errors.

	**Congrats! You just tested an IAM user's access to the console and
modified their privileges as needed.**
