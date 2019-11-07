---
title: "10. Test Assesment"
weight: 10
draft: false
---

Before you schedule vulnerability assessments, you should test the process by running the `StartContinuousAssessment` function. In this test, you trigger a security assessment and monitor it. You then receive an email after the assessment has completed, which shows that vulnerability assessments have been successfully set up.

To start golden AMI vulnerability assessments:

1.  Sign in to the [AWS Management Console](https://console.aws.amazon.com/console/home) and choose **Lambda** in the **Services **menu.
2.  Choose **Functions**. In the **Functions** pane, choose the **StartContinuousAssessment** function.
3.  Choose the **Select a test event** drop-down, and choose **Configure test events**.
4.  On the **Configure test event** page, choose **Create new test event** and specify the event name as test.
5.  Paste the following JSON in the editor box.

    <div class="hide-language">

        {
        "AMIsParamName": "ContinuousAssessmentInput"
        }

    </div>

6.  Choose **Create**. Choose **Test**.

The `StartContinuousAssessment` function runs for approximately five minutes and then displays the following message.  
![Message showing the function has run successfully](https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2017/12/15/KW_1_1217.png "Message showing the function has run successfully")


![](/AMI Inspector Lab/images/Testing01.png)

![](/AMI Inspector Lab/images/Testing02.png)



Next, open Amazon Inspector and monitor the progress of the assessment:

1.  Sign in to the [AWS Management Console](https://console.aws.amazon.com/console/home) and navigate to the [Amazon Inspector console](https://console.aws.amazon.com/inspector/).
2.  On **Dashboard** under **Recent Assessment Runs**, you will see an entry with the status, **Collecting Data**. This status indicates that Amazon Inspector agents are collecting data from instances running your golden AMIs. The agents collect data for an hour and then Amazon Inspector analyzes the collected data.

After Amazon Inspector completes the assessment, the status in the console changes to **Analysis complete**. Amazon Inspector then publishes an SNS message that triggers the `AnalyzeInspectionReports` Lambda function. When `AnalyzeInspectionReports` publishes results, you will receive an email containing consolidated assessment results. You also will be able to see the findings.

To see the findings in Amazon Inspector’s **Findings** section:

1.  Sign in to the [AWS Management Console](https://console.aws.amazon.com/console/home) and navigate to the [Amazon Inspector console](https://console.aws.amazon.com/inspector/home).
2.  In the navigation pane, choose **Assessment Runs**. In the table on the **Amazon Inspector – Assessment Runs** page**,** choose the findings of the latest assessment run.
3.  Choose the settings icon and choose the appropriate tags to see the details of findings, as shown in the following screenshot. The findings also contain information about how you can address each underlying vulnerability.  
    ![Screenshot showing details of findings](https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2017/12/15/KW_2_1217.png "Screenshot showing details of findings")

Having verified that you have successfully set up all components of golden AMI vulnerability assessments, you now will schedule the vulnerability assessments to run on a regular basis to give you continual insight into the health of instances created from your golden AMIs.

### 5.  Set up a CloudWatch Events rule for triggering continuous golden AMI vulnerability assessments

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
