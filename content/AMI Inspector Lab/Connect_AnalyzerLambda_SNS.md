---
title: "7. Subscribe Lambda to SNS Topic"
weight: 7
draft: false
---
Now that we have created our Lambda functions the next thing to do is to connect the `ContinuousAssesmentCompleteTopic` SNS Topic to the `AnalyzeInspectorFindings` Lambda function, leveraging the subscription mechanism. This is to allow `AnalyzeInspectorFindings` to be triggered once Amazon Inspector completes the inspection and to send the notification message to users.

![](/AMI Inspector Lab/images/ContinuousAssesmentLambdaFunction05.png)

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

### Create a Lambda Invoke Permission for the SNS service

In this step we will be **creating a Lambda Invoke Permission** to allow the SNS service to invoke the Lambda function as follows:
    
* Allow the Lambda function to be triggered by the SNS service, `sns.amazonaws.com`
* Attach this permission to the `AnalyzeInspectorFindings` Lambda function you created before

You can refer here for the properties reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-permission.html

Choose how you would like to build the resource from the options below:

<details><summary> **Option 1 - Build the resource manually with step-by-step instructions.**</summary>
<p>    

  * Open your text editor and edit the file named  `GoldenAMIContinuousAssesment.yml`.
  * In the `Resources:` section, add the following:
  * Create a resource named `LambdaInvokePermission` of type `AWS::Lambda::Permission`.
  * In the `Properties` section create a `FunctionName` property and using the `!GetAtt` intrinsic function, reference the `AnalyzeInspectorFindingsLambdaFunction` Arn you created before. [Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html)
  * In the `Properties` section create an `Action` property and specify `lambda:InvokeFunction`
  * In the `Properties` section create a `Principal` property and specify `sns.amazonaws.com`
  * In the `Properties` section create a `SourceArn` property and specify a reference to the `ContinuousAssessmentCompleteTopic` using the `!Ref` intrinsic function

</p>
</details>
        
<details><summary>**Option 2 - CloudFormation template snippet**</summary>

```
      LambdaInvokePermission: 
        Properties: 
          Action: "lambda:InvokeFunction"
          FunctionName: !GetAtt "AnalyzeInspectorFindingsLambdaFunction.Arn"
          Principal: sns.amazonaws.com
          SourceArn: !Ref "ContinuousAssessmentCompleteTopic"
        Type: "AWS::Lambda::Permission"
```
</details>


### Configure an SNS Topic Lambda function Subscription


In this step we will be **creating an SNS Subscription resource for a Lambda function** as follows:
    
* The subscriber must be the Lambda function `AnalyzeInspectorFindings` we created before.
* The SourceArn must be the `ContinuousAssessmentCompleteTopic`

You can refer here for the properties reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-permission.html

Choose how you would like to build the resource from the options below:

<details><summary> **Option 1 - Build the resource manually with step-by-step instructions.**</summary>
<p>    

  * Open your text editor, and edit the file named  `GoldenAMIContinuousAssesment.yml`.
  * Add the following resource to the `Resources:` section:
  * Create a resource named `ContinuousAssessmentCompleteTopicSubscription` of type `AWS::SNS::Subscription`.
  * In the `Properties` section, create an `Endpoint` property and using the `!GetAtt` intrinsic function, reference the `AnalyzeInspectorFindingsLambdaFunction` Arn you created before. [Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html)
  * In the `Properties` section create a `Protocol` property and specify `lambda`.
  * In the `Properties` section create a `SourceArn` property and specify a reference to the `ContinuousAssessmentCompleteTopic` using the `!Ref` intrinsic function.

<p>
</details>
        
<details><summary>**Option 2 - CloudFormation template snippet**</summary>

**NOTE** The snippet below must be specified within the `Resources:` section of the CloudFormation template.

```
      ContinuousAssessmentCompleteTopicSubscription: 
        Properties: 
          Endpoint: !GetAtt "AnalyzeInspectorFindingsLambdaFunction.Arn"
          Protocol: lambda
          TopicArn: !Ref "ContinuousAssessmentCompleteTopic"
        Type: "AWS::SNS::Subscription"
```
</details>

### Deploy the CloudFormation template

Now that you've updated the template, it's time to update the stack as well. 
    
To do that please follow the [Update a Stack's Template (Console)](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-get-template.html#using-cfn-updating-stacks-get-stack.CON) guide.

Once you've launched your stack review the `Resources` Tab of the launch stack to identify the resources that have been created for you.

**Hey, Hey! Well done!** We have created all of the main resources for our solution. Also, you've packaged this inside a CloudFormation template. Next time you need to deploy this solution, it will take you minutes, rather than the time you have spent so far in building it.

Let's move to next page to test our solution.