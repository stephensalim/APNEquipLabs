---
title: "2. Tag your Golden AMI"
weight: 2
draft: false
---

  ![](/AMI Inspector Lab/images/TagGoldAMI.png)


Before we start, you must set this solution in the AWS Region where you build your golden AMIs and have [Amazon Inspector](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_supported_os_regions.html#inspector_supported-regions) available. If you don't have any preferance, please choose Sydney Region ( ap-southeast-2) as this lab has been tested in this region.

In this solution you will search assessment findings based on golden AMI tags after Amazon Inspector completes an assessment.

To tag a golden AMI by using the AWS Management Console:

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

Now that you have tagged your golden AMIs, you need to create golden AMI metadata, which will be red by the `StartContinuousAssessment` Lambda function to initiate vulnerability assessments. You will store the golden AMI metadata in the Systems Manager Parameter Store.

