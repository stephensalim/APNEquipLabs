---
title: "2. Create Stack"
weight: 2
draft: false
---

Copy and paste the **Sample CloudFormation Template** from the previous section to a text file called Sample_VPC_CloudFormation.yml

Log into the **AWS Console**, and click on **CloudFormation** and below
screen will open:

![](/CloudFormation Lab/images/media/image2.png)

Now click on **Create new stack** and browse your initial template to
against choose a template option :

![](/CloudFormation Lab/images/media/image3.png)

Click **Next** and give stack name. Make sure your stack name should be
unique to your account. Leave all other option as default

![](/CloudFormation Lab/images/media/image4.png)

Click **Next**, here you can define a tag for the stack, IAM Role and
other advance option like termination protection and rollback trigger.
For this lab we will leave this as it and click to **Next** again, where
you will have the opportunity to review your stack settings:

![](/CloudFormation Lab/images/media/image5.png)

Now Click on **Create** and you will notice your stack creation started
with status **CREATE\_IN\_PROGRESS.** Explore the **Events** tab where
you can see the progress as your stack get created.

![](/CloudFormation Lab/images/media/image6.png)

While you are waiting to explore all other tabs like **Template** tab to
review your template and **Parameters** tab to see parameter value. You
will also notice that **Outputs tab** is empty and you are going to
modify your template to show values in Outputs tab.

![](/CloudFormation Lab/images/media/image7.png)

![](/CloudFormation Lab/images/media/image8.png)

Once stack status changes to **CREATE\_COMPLETE,** you can visit
**Resources** tab to see all the resources got created by this AWS
CloudFormation template.

![](/CloudFormation Lab/images/media/image9.png)

You can click on Amazon S3 bucket link shown in **Resources** tab and
explore the bucket.Also, go to VPC from the console and explore
different resources got created from AWS CloudFormation stack.

![](/CloudFormation Lab/images/media/image10.png)
