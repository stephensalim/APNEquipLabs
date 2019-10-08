---
title: "2. Tag your Golden AMI"
weight: 2
draft: false
---

To deploy this solution, you must set it up in the AWS Region where you build your golden AMIs. If that AWS Region does not [support Amazon Inspector](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_supported_os_regions.html#inspector_supported-regions), at the end of your continuous integration pipeline, you can copy your AMIs to an AWS Region where Amazon Inspector assessments are supported. 

You can search assessment findings based on golden AMI tags after Amazon Inspector completes an assessment.

To tag a golden AMI by using the AWS Management Console:

1.  Sign in to the [AWS Management Console](https://console.aws.amazon.com/console/home) and then navigate to the [EC2 console](https://console.aws.amazon.com/ec2/v2/home).
2.  In the navigation page, choose Instances then click **Launch Instance**.

    ![](/AMI Inspector Lab/images/img001.png)

3.  Spot the **Amazon Linux 2 AMI (HVM)** and take note of the AMI ID.

    ![](/AMI Inspector Lab/images/img002.png)

4.  In the navigation pane, choose **AMIs**.
5.  In the search ensure the setting is set to **Public Images**.
6.  In teh search field, search for AMI of the AMI you noted in step 3

    ![](/AMI Inspector Lab/images/img003.png)

3.  Choose your AMI from the list, and then choose **Actions** > **Add/Edit Tags**.
4.  Choose **Create Tag.** In the **Key** column, type `app-name`. In the **Value** column, type your application name. Following the same steps, create the `app-version` and `app-environment` tags. Choose **Save**.

    ![](/AMI Inspector Lab/images/img004.png)

Now that you have tagged your golden AMIs, you need to create golden AMI metadata, which will be read by the `StartContinuousAssessment` function to initiate vulnerability assessments. You will store the golden AMI metadata in the Systems Manager Parameter Store.

