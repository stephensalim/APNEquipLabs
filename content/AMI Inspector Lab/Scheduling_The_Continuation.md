---
title: "11. Configure CloudWatch events"
weight: 11
draft: false
---

**Set up a CloudWatch Events rule for triggering continuous golden AMI vulnerability assessments**

![](/AMI Inspector Lab/images/CloudWatchRule.png)

The last step is to create a CloudWatch Events rule to schedule the execution of the vulnerability assessments on a daily or weekly basis.

To set up a CloudWatch Events rule:

1.  Sign in to the [AWS Management Console](https://console.aws.amazon.com/console/home) and navigate to the [CloudWatch console](https://console.aws.amazon.com/cloudwatch/).
2.  In the navigation pane, choose **Rules** > **Create rule**.
3.  On the **Event Source** page, choose **Schedule**. Choose **Fixed rate of **and specify the interval (for example, 1 day).
4.  For **Targets**, choose **Add target **and then choose **Lambda function**.
5.  For **Function**, choose the **StartContinuousAssessment** function.
6.  Choose **Configure Input**.
7.  Choose **Constant (JSON text)**.
8.  In the box, paste the following JSON code.

    <div class="hide-language">

        {
             "AMIsParamName": "ContinuousAssessmentInput"
        }

    </div>

9.  Choose **Configure details**.
10.  For **Rule definition**, type `ContinuousGoldenAMIAssessmentTrigger` for the name, and type as the description, `This rule triggers the continuous golden AMI vulnerability assessment process`.
11.  Choose **Create rule**.

The vulnerability assessments are executed on the first occurrence of the schedule you chose while setting up the CloudWatch Events rule. After the vulnerability assessment **is** executed, you will receive an email to indicate that your continuous golden AMI vulnerability assessments are set up.

### Summary

To get visibility into the security of your EC2 instances created from your golden AMIs, it is important that you perform security assessments of your golden AMIs on a regular basis. In this blog post, I have demonstrated how to set up vulnerability assessments, and the results of these continuous golden AMI vulnerability assessments can help you keep your environment up to date with security patches. To learn how to patch your golden AMIs, see [Streamline AMI Maintenance and Patching Using Amazon EC2 Systems Manager](https://aws.amazon.com/blogs/aws/streamline-ami-maintenance-and-patching-using-amazon-ec2-systems-manager-automation/).

If you have comments about this blog post, submit them in the “Comments” section below. If you have questions about implementing the solution in this post, start a new thread on the [Amazon Inspector forum](https://forums.aws.amazon.com/forum.jspa?forumID=205) or [contact AWS Support](https://console.aws.amazon.com/support/home).
