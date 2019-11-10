---
title: "9. Configure CloudWatch Event Rule"
weight: 9
draft: false
---

This will be the last piece of the puzzle!
In this section of the Lab we will be extending our CloudFormation template to add a CloudWatch Events scheduler, which will allow Lambda functions to regularly assess Golden AMIs.

![](/AMI Inspector Lab/images/CloudWatchRule.png)

---

**IMPORTANT NOTE:**

In the following steps you will need to construct your CloudFormation template in YAML format.
YAML allows you to put comments in the template by placing a `#` in front of the comment line.
YAML is an indentation sensitive syntax, therefore make sure to specify the key and values at the right indent level in the template.

When building a CloudFormation template, it is always recommended to refer back to the extensive CloudFormation documentation.
In the following steps we will be providing the template snippets for the specific resource for a smooth lab experience.
Alternatively, we will also be providing high level instructions on how to construct the template if you choose to build them manually. We will also provide reference guides and examples from the documentation to help you. 
    
The purpose of this lab is get accustomed to exploring CloudFormation documentation and syntax.
That being said, if at any point you get stuck, do reach out to the lab support team, or consult the reference template from the solution overview.

---

### Set up the CloudWatch Events rule 

In this step we will be **creating CloudWatch Events Rule** with the following characteristics:
    
* Executes 6 AM daily.
* The Target is the Arn of `StartContinuousAssessment` Lambda function.
* Injects the input value of `'{"AMIsParamName": "ContinuousAssessmentInput"}'` into the Lambda function to kick off the process.

You can refer here for the properties reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-events-rule.html


Choose how you would like to build the resource from the options below:

<details><summary> **Option 1 - Build the resource manually with step-by-step instructions.**</summary>
<p>    

* Open your text editor, and create a file named `GoldenAMIContinuousAssesment.yml`.
* Create a `Resource:` template section. [Reference](https://docs.aws.amazon.com/en_pv/AWSCloudFormation/latest/UserGuide/template-anatomy.html) 
* Create a resource named `ContinuousGoldenAMIAssessmentTrigger` of type `AWS::Events::Rule`.
* In the `Properties` section add a `Name` property and specify `ContinuousGoldenAMIAssessmentTrigger` as its value.
* In the `Properties` section add a `ScheduleExpression` property and specify `"cron(0 6 * * ? *)"` as its value.
* In the `Properties` section add a `State` property and specify `ENABLED` as its value.
* In the `Properties` section add a `Targets` property and using the `!GetAtt` intrinsic function, reference the `StartContinuousAssessment` Arn you created in the previous step. [Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html)
* In same section as your `Targets` entries, add `Inputs` and specify `'{"AMIsParamName": "ContinuousAssessmentInput"}'` as its value.
* In same section as your `Targets` entries, add `Id` and specify `ContinuousGoldenAMIAssessmentTrigger` as its value.

<p>
</details>

<details><summary>**Option 2 - CloudFormation template snippet**</summary>

**NOTE** The snippet below must be specified within the `Resources:` section of the CloudFormation template.

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

**CONGRATULATIONS!!!!** **~~~ \\[^0^]/ ~~~** 

You have completed the Lab. Hopefully by now you have captured some insights on how to package your solutions and make them repeatable. Good luck!
