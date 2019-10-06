---
title: "4. Update Stack"
weight: 4
draft: false
---

Once you modified your existing template , you can use **Update Stack**
option to update your stack. To update select your Stack and click on
**Actions drop down and you will find Update Stack** option**.**

![](/CloudFormation Lab/images/media/image11.png)

In **Update**, Stack screen select browse your updated template. If you
have not figured out a solution yet Follow instruction in appendix
section to get template

*Lab\_Solution\_CloudFormation\_Module\_General\_ImmersionDay.json*.

![](/CloudFormation Lab/images/media/image12.png)

Now remaining steps are same as you followed in Create stack. Click Next
couple of time and you will land up to review summary screen, where you
need to click on **Update** button :

![](/CloudFormation Lab/images/media/image13.png)

Now you will find your stack status changed to **UPDATE\_IN\_PROGRESS**
and **Events** tab showing the activity performed using update stack.

![](/CloudFormation Lab/images/media/image14.png)

Once stack status changed to UPDATE\_COMPLETE status, you can browse to
**Outputs** tab and find out our changes has reflected now Outputs tab
has four values compare to earlier it was empty :

![](/CloudFormation Lab/images/media/image15.png)

Also, click on **CloudFormation** icon on the right top corner of the
screen and select **Exports** option**,** you will find two exported
value shown in here which can be utilized for cross-stack reference.

![](/CloudFormation Lab/images/media/image16.png)

To create a cross-stack reference, use the **Export** output field to
flag the value of a resource-output for export. Then, use the **Fn::
ImportValue** intrinsic function to import the value.
