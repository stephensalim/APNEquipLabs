---
title: "2. Tagging the Golden AMI"
weight: 2
draft: false
---

Before we begin, the first thing we need to have is a **Golden AMI**, the subject of our continuous assessment solution.
To do this, you can create your own Golden AMI from an EC2 instance as described in this [guide](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/tkv-create-ami-from-instance.html). Alternatively, to keep it simple, you can choose to use an Amazon Public AMI as your Golden AMI. But please note, AWS may retract access to the public AMI in the future, and should this happen, it will no longer be accessible to you. Therefore, as a best practice, it is always recommended to use an AMI that you have created yourself as your **Golden AMI**. 
But, for simplicity, for this lab we will be using an Amazon Public AMI as a Golden AMI. 

In this part of the lab we will be **tagging our Golden AMI** to put context around what application, version and environment the AMI is based on. These tags will later be transferred to the temporary EC2 instance during assessment, and to the Amazon Inspector results in order to provide context for the findings. 

![](/AMI Inspector Lab/images/TagGoldAMI.png)

Follow the instructions below in order to **tag your Golden AMI** using the AWS Management Console:

1.  Sign in to the [AWS Management Console](https://console.aws.amazon.com/console/home) and then navigate to the [EC2 console](https://console.aws.amazon.com/ec2/v2/home).
2.  In the navigation pane, choose **Instances** then click **Launch Instance**.

    ![](/AMI Inspector Lab/images/img001.png)

3.  Find the **Amazon Linux 2 AMI (HVM)** and take note of the AMI ID.

    ![](/AMI Inspector Lab/images/img002.png)

4.  Click **Cancel and Exit**. _(Ideally you should use your own golden image, but for the purpose of this lab we will be using an AWS public AMI as the golden AMI.)_

5.  In the navigation pane, choose **AMIs**.
6.  In the search ensure the setting is set to **Public Images**.
7.  In the search field, search for AMI of the AMI you noted in step 3

    ![](/AMI Inspector Lab/images/img003.png)

8.  Choose your AMI from the list, and then choose **Actions** > **Add/Edit Tags**.
9.  Choose **Create Tag.** In the **Key** column, type `app-name`. In the **Value** column, type your application name. Following the same steps, create the `app-version` and `app-environment` tags. Choose **Save**.

    ![](/AMI Inspector Lab/images/img004.png)

Once you've completed these steps, you now have tagged your Golden AMI with contextual information. Let's move on to the next step.

**~[`^0^]~**