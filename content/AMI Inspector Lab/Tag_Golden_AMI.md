---
title: "2. Tag your Golden AMI"
weight: 2
draft: false
---

  ![](/AMI Inspector Lab/images/TagGoldAMI.png)


Before we start anything in this solution the first thing we need to do is have the Subject Golden AMI. Now to do this, you can either create your own AMI basing from existing instance running AMI Amazon Linux 2 by following this [Guide](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/tkv-create-ami-from-instance.html) or you can just use the Amazon public AMI. The following instructions will assume you are using Amazon public AMI.

---

**IMPORTANT NOTE:** 

You must set this solution in the AWS Region where you build your golden AMIs and have [Amazon Inspector](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_supported_os_regions.html#inspector_supported-regions) available. If you don't have any preferance, please choose Sydney Region **( ap-southeast-2)** as this lab has been tested in this region. In this solution you will search assessment findings based on golden AMI tags after Amazon Inspector completes an assessment.

---

Follow below instructions to tag a golden AMI by using the AWS Management Console:

1.  Sign in to the [AWS Management Console](https://console.aws.amazon.com/console/home) and then navigate to the [EC2 console](https://console.aws.amazon.com/ec2/v2/home).
2.  In the navigation page, choose Instances then click **Launch Instance**.

    ![](/AMI Inspector Lab/images/img001.png)

3.  Spot the **Amazon Linux 2 AMI (HVM)** and take note of the AMI ID.

    ![](/AMI Inspector Lab/images/img002.png)

4.  Click **Cancel and Exit** (Ideally you could use your own golden image but for the purpose of this lab we will be using AWS public AMI as the golden AMI.)

5.  In the navigation pane, choose **AMIs**.
6.  In the search ensure the setting is set to **Public Images**.
7.  In the search field, search for AMI of the AMI you noted in step 3

    ![](/AMI Inspector Lab/images/img003.png)

8.  Choose your AMI from the list, and then choose **Actions** > **Add/Edit Tags**.
9.  Choose **Create Tag.** In the **Key** column, type `app-name`. In the **Value** column, type your application name. Following the same steps, create the `app-version` and `app-environment` tags. Choose **Save**.

    ![](/AMI Inspector Lab/images/img004.png)

Once you've completed these steps, you now have tagged your Golden AMI with information so the context of the AMI can be easily spotted.
The next thing to do is to create the Golden AMI metadata, and store them inside [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html) This will allow our solution to pull in instructions and execute the activity towards the AMI we created. Go to next step to create this metadata.
