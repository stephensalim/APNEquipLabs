---
title: "9. Configure CloudWatch events"
weight: 9
draft: false
---

This will be the last piece of the puzzle.
In this part of the Lab we will be extending our CloudFormation template to add a CloudWatch Events scheduler to allow our Lambda Functions to continusously asses Golden AMIs.

![](/AMI Inspector Lab/images/CloudWatchRule.png)

1. **Set up a CloudWatch Events rule for triggering continuous golden AMI vulnerability assessments**


    Open your notepad / text editor, create a file named `GoldenAMIContinuousAssesment.yml`.

    ---

    **IMPORTANT NOTE:**
    In the following steps you will be constructing your CloudFormation template in YML format.
    YML format allows you to put comments in the template by placing in # in front of the line, so it's quite handy.
    On the flip side however, it is indent sensitive, so make sure you specify the Key and values at the right level of the indentation.

    Practice makes perfect, therefore when building CloudFormation template in this lab, we will be providing a high level instruction on how to construct the template, along with the reference guide in the public documentation. The intent is so that you could get used to exploring the public documentation and get acustomed with the syntax.

    Having said that, if you are completely stuck, don't hesitate to get help from Lab instructor or, take a peek at the **SOLUTION** section if you have to.

    ---

    To find information about the properties of the resource refer to this doc: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-events-rule.html

   * Create a `Resource:` template section [Reference](https://docs.aws.amazon.com/en_pv/AWSCloudFormation/latest/UserGuide/template-anatomy.html) 
    * Create a resource named `ContinuousGoldenAMIAssessmentTrigger` of type `AWS::Events::Rule`.
    * In the `Properties` section add `Name` property and specify `ContinuousGoldenAMIAssessmentTrigger` as it's value.
    * In the `Properties` section add `ScheduleExpression` property and specify `"cron(0 6 * * ? *)"` as it's value.
    * In the `Properties` secion add an `State` and specify `ENABLED` as it's value.
    * In the `Properties` secion add an `Targets` property and using the !GetAtt intrinsic function reference the `StartContinuousAssessment` Arn you created in previous step Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html
    * In same section as your `Targets` entries and specify `Inputs` and specify `'{"AMIsParamName": "ContinuousAssessmentInput"}'` as it's value.
       
<details><summary> **ARE YOU STUCK ? :(** - It's OK CLICK HERE to see the solution</summary>

**READ >>** Below snippet must be specified within `Resources:` section of the cloudformation template.

```
  ContinuousGoldenAMIAssessmentTrigger:
    Type: AWS::Events::Rule
    Properties:
      Name: ContinuousGoldenAMIAssessmentTrigger
      ScheduleExpression: "cron(0 6 * * ? *)"
      State: ENABLED
      Targets:
        -
          Arn:
            Fn::GetAtt:
              - "StartContinuousAssessment"
              - "Arn"
          Input: '{"AMIsParamName": "ContinuousAssessmentInput"}'
```
</details>


The vulnerability assessments are executed on the first occurrence of the schedule you chose while setting up the CloudWatch Events rule. After the vulnerability assessment **is** executed, you will receive an email to indicate that your continuous golden AMI vulnerability assessments are set up.

### Summary

To get visibility into the security of your EC2 instances created from your golden AMIs, it is important that you perform security assessments of your golden AMIs on a regular basis. In this blog post, I have demonstrated how to set up vulnerability assessments, and the results of these continuous golden AMI vulnerability assessments can help you keep your environment up to date with security patches. To learn how to patch your golden AMIs, see [Streamline AMI Maintenance and Patching Using Amazon EC2 Systems Manager](https://aws.amazon.com/blogs/aws/streamline-ami-maintenance-and-patching-using-amazon-ec2-systems-manager-automation/).

If you have comments about this blog post, submit them in the “Comments” section below. If you have questions about implementing the solution in this post, start a new thread on the [Amazon Inspector forum](https://forums.aws.amazon.com/forum.jspa?forumID=205) or [contact AWS Support](https://console.aws.amazon.com/support/home).
