---
title: "2. Create the Stack"
weight: 2
draft: false
---

Copy and paste the **Sample CloudFormation Template** from the previous section to a text file called Sample_VPC_CloudFormation.yml

Log into the **AWS Console**, and click on **CloudFormation** and the following screen will be displayed:

![](/CloudFormation Lab/images/media/image2.png)

Now click on **Create new stack**, choose the option to upload a template, 
and browse to your initial template:

![](/CloudFormation Lab/images/media/image3.png)

Click **Next** and provide a stack name. Make sure your stack name is unique within
your account. Leave all other options as defaults:

![](/CloudFormation Lab/images/media/image4.png)

Click **Next**, from here you can define tags for the stack, IAM Roles and
other advanced options like termination protection and rollback triggers.
For this lab we will leave this as it is, and click **Next** again, where
you will have the opportunity to review your stack settings:

![](/CloudFormation Lab/images/media/image5.png)

Now Click on **Create** and you will notice your stack creation has started
with a status of **CREATE\_IN\_PROGRESS**. Explore the **Events** tab where
you can see the progress as each resource in your stack gets created.

![](/CloudFormation Lab/images/media/image6.png)

While you are waiting for the stack creation to complete, be sure to explore
the other tabs, such as the **Template** tab to
review your template and the **Parameters** tab to see parameter values. You
will also notice that **Outputs** tab is empty. In the next steps you will modify
your template to produce values in **Outputs** tab.

![](/CloudFormation Lab/images/media/image8.png)

![](/CloudFormation Lab/images/media/image7.png)

Once stack status changes to **CREATE\_COMPLETE,** you can visit
the **Resources** tab to see all of the resources that were created by 
this AWS CloudFormation template.

![](/CloudFormation Lab/images/media/image9.png)

You can click on the Amazon S3 bucket link shown in **Resources** tab and
explore the bucket. Also, navigate to the **VPC** service from the console, and explore
the different resources that were created as part of the AWS CloudFormation stack.

![](/CloudFormation Lab/images/media/image10.png)
