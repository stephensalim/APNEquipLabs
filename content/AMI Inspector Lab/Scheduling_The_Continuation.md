---
title: "9. Configure CloudWatch Event Rule"
weight: 9
draft: false
---

This will be the last piece of the puzzle !
In this part of the Lab we will be extending our CloudFormation template to add a CloudWatch Events scheduler allowing our Lambda Functions to regularly asses Golden AMIs.

![](/AMI Inspector Lab/images/CloudWatchRule.png)

---

**IMPORTANT NOTE:**

In the following steps you will need to construct your CloudFormation template in YAML format.
YAML format allows you to put comments in the template by placing in # in front of the line.
YAML format is indent sensitive syntax, therefore make sure to specify the key and values at the right indent level in the template.

When building a cloudformation template, it is recommended to always refer back to the extensive CloudFormation documentation.
In the folloing steps for smooth lab experience, we will be providing the template snippet for the specific resource.
Alternatively, we will also be providing a high level instruction on how to construct the template if you coose to build them manually. We will also provide reference guide / example from public documentation to help you. 
    
The purpose of this is get acustomed towards exploring CloudFormation documentation and syntax.
That being said, at any point in time you are stuck reach out to the lab support team.

---

### Set up a CloudWatch Events rule 

In this step we will be **creating cloudwatch rule** with below criteria :
    
* Executes 6 AM daily.
* Target is the ARN of StartContinuousAssessment Lambda Function.
* Inject input value of `'{"AMIsParamName": "ContinuousAssessmentInput"}'` into the Lambda Function to kick off the process.

You can refer here for properties reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-events-rule.html


Choose one of the option below on how you want to build the resource.
 
<details><summary>**Option 1 - Build the resouce manually with step by step instructions.**</summary>
<p>    

* Open your notepad / text editor, create a file named `GoldenAMIContinuousAssesment.yml`.
* Create a `Resource:` template section [Reference](https://docs.aws.amazon.com/en_pv/AWSCloudFormation/latest/UserGuide/template-anatomy.html) 
* Create a resource named `ContinuousGoldenAMIAssessmentTrigger` of type `AWS::Events::Rule`.
* In the `Properties` section add `Name` property and specify `ContinuousGoldenAMIAssessmentTrigger` as it's value.
* In the `Properties` section add `ScheduleExpression` property and specify `"cron(0 6 * * ? *)"` as it's value.
* In the `Properties` secion add an `State` and specify `ENABLED` as it's value.
* In the `Properties` secion add an `Targets` property and using the !GetAtt intrinsic function reference the `StartContinuousAssessment` Arn you created in previous step Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html
* In same section as your `Targets` entries and specify `Inputs` and specify `'{"AMIsParamName": "ContinuousAssessmentInput"}'` as it's value.
* In same section as your `Targets` entries and specify `Id` and specify `ContinuousGoldenAMIAssessmentTrigger` as it's value.

<p>
</details>

<details><summary>**Option 2 - CloudFormation template snippet**</summary>

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

**CONGRATS !** **~~~ \[^0^]/ ~~~** 

You have completed the Lab. Hopefully by now you have caputred some insights on how to package your solutions and make it repeatable.
