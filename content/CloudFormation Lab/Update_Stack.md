---
title: "4. Update the Stack"
weight: 4
draft: false
---

Once you have modified your template, you can use the **Update Stack**
option to update your stack. To start the update, your stack and click on
**Actions** drop down and you will find the **Update Stack** option. 
(Your console version may appear different from the image below, if so,
after selecting your stack, just click **Update**.)

![](/CloudFormation Lab/images/media/image11.png)

In the **Update** stack screen, select the option to upload a new 
template. If you have not yet created a solution, download the 
reference template below.

[Reference template](/downloads/CloudFormation_Lab_Reference_Template.yml)

![](/CloudFormation Lab/images/media/image12.png)

The remaining steps are same as you followed when creating your stack. 
Click **Next** until you come to the review summary screen, where you
need to click on the **Update** button :

![](/CloudFormation Lab/images/media/image13.png)

You should see your stack's status change to **UPDATE\_IN\_PROGRESS**
and the **Events** tab will show the activities performed while updating your stack.

![](/CloudFormation Lab/images/media/image14.png)

Once your stack's status has changed to **UPDATE\_COMPLETE**, you can browse to the
**Outputs** tab and see how your changes have been applied. The four values you have
configured in the template's **Outputs** section are now reflected in the **Outputs** tab, where
previously it was empty:

![](/CloudFormation Lab/images/media/image15.png)

Next, click on **CloudFormation** icon on the right top corner of the
screen, and select **Exports** option** (or if your console has been updated, click on **Exports** in left column),
and you will find the two values you have exported from your stack which can be utilized as cross-stack references.

![](/CloudFormation Lab/images/media/image16.png)

To create a cross-stack reference, in a second stack you can use the field in the **Exports** output and the **Fn::
ImportValue** intrinsic function to import the value you have exported from the first stack.

See the AWS CloudFormation documentation for more information:

<https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-importvalue.html>

