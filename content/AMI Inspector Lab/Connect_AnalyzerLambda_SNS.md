---
title: "7. Subscribe Lambda to SNS Topic"
weight: 7
draft: false
---
Now that we have created our Lambda Functions the next thing to do is to connect the `ContinuousAssesmentCompleteTopic` SNS Topic to the `AnalyzeInspectorFindings` Lambda function, levaraging the subscribtion mechanism. This is to basically allow `AnalyzeInspectorFindings` to be triggered once Amazon Inspector completes the inspection and send the message notification to users.

![](/AMI Inspector Lab/images/ContinuousAssesmentLambdaFunction05.png)

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

### Create Lambda Permission to allow SNS service to invoke.

In this step we will be **creating a Lambda Permission** with below criteria :
    
* Allows the Lambda Function to be triggered via SNS service `sns.amazonaws.com`
* Attach this permission to the `AnalyzeInspectorFindings` Lambda Function you created before.

You can refer here for properties reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-permission.html

Choose one of the option below on how you want to build the resource.
 
<details><summary>**Option 1 - Build the resouce manually with step by step instructions.**</summary>
<p>    

    * Open your notepad / text editor, edit the file named  `GoldenAMIContinuousAssesment.yml`.
    * Right under the previous resource, still inside the `Resources:` section do the following.
    * Create a resource named `LambdaInvokePermission` of type `AWS::Lambda::Permission`.
    * In the `Properties` section create a `FunctionName` property and using the !GetAtt intrinsic function reference the `AnalyzeInspectorFindingsLambdaFunction` Arn you created before Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html
    * In the `Properties` section create a `Action` property and specify `lambda:InvokeFunction`
    * In the `Properties` section create a `Principal` property and specify `sns.amazonaws.com`
    * In the `Properties` section create a `SourceArn` property and specify a reference to the `ContinuousAssessmentCompleteTopic` using !Ref intrinsic function

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


### Configure SNS Topic Lambda Function Subscriber


In this step we will be **a SNS Subscriber resource** with below criteria :
    
* The Subscriber is Lambda function `AnalyzeInspectorFindings` we created before.
* Set the SourceArn as the `ContinuousAssessmentCompleteTopic`

You can refer here for properties reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-sns-subscription.html

Choose one of the option below on how you want to build the resource.
 
<details><summary>**Option 1 - Build the resouce manually with step by step instructions.**</summary>
<p>    

  * Open your notepad / text editor, edit the file named  `GoldenAMIContinuousAssesment.yml`.
  * Right under the previous resource, still inside the `Resources:` section do the following.
  * Create a resource named `ContinuousAssessmentCompleteTopicSubscription` of type `AWS::SNS::Subscription`.
  * In the `Properties` section create a `Endpoint` property and using the !GetAtt intrinsic function reference the `AnalyzeInspectorFindingsLambdaFunction` Arn you created before. Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html
  * In the `Properties` section create a `Protocol` property and specify `lambda`.
  * In the `Properties` section create a `SourceArn` property and specify a reference to the `ContinuousAssessmentCompleteTopic` using !Ref intrinsic function.

<p>
</details>
        
<details><summary>**Option 2 - CloudFormation template snippet**</summary>

**READ >>** Below snippet must be specified within `Resources:` section of the cloudformation template.

```
      ContinuousAssessmentCompleteTopicSubscription: 
        Properties: 
          Endpoint: !GetAtt "AnalyzeInspectorFindingsLambdaFunction.Arn"
          Protocol: lambda
          TopicArn: !Ref "ContinuousAssessmentCompleteTopic"
        Type: "AWS::SNS::Subscription"
```
</details>

### Deploys the CloudFormation Template

Now that you've construct the template, it's time to update the stack, do do that please follow the [Update a Stack's Template (Console)](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-get-template.html#using-cfn-updating-stacks-get-stack.CON) guide to create your stack.

Specify Stack `GoldenAMIContinuousAssesment` as the stack name for simplicity.

Once you've launched your stack review the `Resources` Tab of the launch stack to identify the resouce it's created.

**Hey Hey ! Well done !** We have created all of the main resources for our solution. on top of that you've packaged this inside CloudFormation. Next time you need to deploy this solution, it will just take you minutes.

Let's move to next page to Test our solutions