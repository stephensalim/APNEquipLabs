---
title: "9. Configure CloudWatch events"
weight: 9
draft: false
---

This will be the last piece of the puzzle.
In this part of the Lab we will be extending our CloudFormation template to add a CloudWatch Events scheduler to allow our Lambda Functions to continusously asses Golden AMIs.

![](/AMI Inspector Lab/images/CloudWatchRule.png)

1. **Set up a CloudWatch Events rule for triggering continuous golden AMI vulnerability assessments**

    ---

    **IMPORTANT NOTE:**

    In the following steps you will need to construct your CloudFormation template in YAML format.
    YAML format allows you to put comments in the template by placing in # in front of the line, so it's quite handy.
    On the flip side however, it is indent sensitive, so make sure you specify the Key and values at the right level of indentation.

    Practice makes perfect, therefore when building the template, we will be providing a high level instruction on how to construct it.
    We will also provide reference guide / example from public documentation to help you. 
    The purpose of this is so that you could get used to exploring CloudFormation documentation and get acustomed with the syntax.

    However if you are completely stuck, don't hesitate to get help from Lab instructor or take a peek at the **ARE YOU STUCK?** section to peek on the solution.

    ---

    Reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-events-rule.html

    * Open your notepad / text editor, create a file named `GoldenAMIContinuousAssesment.yml`.
    * Create a `Resource:` template section [Reference](https://docs.aws.amazon.com/en_pv/AWSCloudFormation/latest/UserGuide/template-anatomy.html) 
    * Create a resource named `ContinuousGoldenAMIAssessmentTrigger` of type `AWS::Events::Rule`.
    * In the `Properties` section add `Name` property and specify `ContinuousGoldenAMIAssessmentTrigger` as it's value.
    * In the `Properties` section add `ScheduleExpression` property and specify `"cron(0 6 * * ? *)"` as it's value.
    * In the `Properties` secion add an `State` and specify `ENABLED` as it's value.
    * In the `Properties` secion add an `Targets` property and using the !GetAtt intrinsic function reference the `StartContinuousAssessment` Arn you created in previous step Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html
    * In same section as your `Targets` entries and specify `Inputs` and specify `'{"AMIsParamName": "ContinuousAssessmentInput"}'` as it's value.
    * In same section as your `Targets` entries and specify `Id` and specify `ContinuousGoldenAMIAssessmentTrigger` as it's value.
    
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
              - "StartContinuousAssessmentLambdaFunction"
              - "Arn"
          Input: '{"AMIsParamName": "ContinuousAssessmentInput"}'
          Id: ContinuousGoldenAMIAssessmentTrigger
```
</details>

CONGRATS ! You have completed the Lab.
Hopefully by now you have caputred some insights on how to package your solutions and make it repeatable.
