---
title: "2. Tagging the Golden AMI"
weight: 2
draft: false
---

Before we begin, the first thing we need have is a **Golden AMI**. the subject of our Continous Assessment solution.
To do this, you can create your own Golden AMI from EC2 instance as discribed in this [Guide](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/tkv-create-ami-from-instance.html). Alternatively, to keep it simple, you can choose to use Amazon Public AMI as your Golden AMI. But please note that AWS may retract access to the AMI in the future, and when this happen, you wont have access to it anymore. As a best practice, it is always recommended to use your created own AMI as your **Golden AMI**. 
But for the purpose of simplicity in this lab, we will be using Amazon Public AMI as your Golden AMI. 

In this part of the lab we will be **Tagging our Golden AMI** to put context around what application / version / environment the AMI is based on. These tag will later be transfered to the temporary EC2 instance during assesment and to the Amazon Inspector result, to provide AMI context around the findings. 

![](/AMI Inspector Lab/images/TagGoldAMI.png)

Follow below instructions to **Tag the Golden AMI** using the AWS Management Console:

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

Once you've completed these steps, you now have tagged your Golden AMI with contextual information. Let's move on to the next step.

**~[`^0^]~**