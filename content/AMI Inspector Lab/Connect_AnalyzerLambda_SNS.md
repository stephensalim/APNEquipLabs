---
title: "7. Connect Analyzer to Topic"
weight: 7
draft: false
---

Now that we have created our Lambda Functions the next thing to do is to connect the ContinuousAssesmentCompleteTopic SNS Topic to the `AnalyzeInspectorFindings` Lambda function, levaraging the subscribtion mechanism. This is to basically allow `AnalyzeInspectorFindings` to be triggered once Amazon Inspector completes inspection and send the message notification

![](/AMI Inspector Lab/images/ContinuousAssesmentLambdaFunction05.png)

1. **Create Lambda Permission to allow SNS service to invoke.**.

    Open your notepad / text editor, edit the file named  `GoldenAMIContinuousAssesment.yml`.

    ---

    **IMPORTANT NOTE:**
    In the following steps you will be constructing your CloudFormation template in YML format.
    YML format allows you to put comments in the template by placing in # in front of the line, so it's quite handy.
    On the flip side however, it is indent sensitive, so make sure you specify the Key and values at the right level of the indentation.

    Practice makes perfect, therefore when building CloudFormation template in this lab, we will be providing a high level instruction on how to construct the template, along with the reference guide in the public documentation. The intent is so that you could get used to exploring the public documentation and get acustomed with the syntax.

    Having said that, if you are completely stuck, don't hesitate to get help from Lab instructor or, take a peek at the **SOLUTION** section if you have to.

    ---

    To find information about the properties of the resource refer to this doc: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-permission.html

    * Right under the previous resource, still inside the `Resources:` section do the following.
    * Create a resource named `LambdaInvokePermission` of type `AWS::Lambda::Permission`.
    * In the `Properties` section create a `FunctionName` property and using the !GetAtt intrinsic function reference the `AnalyzeInspectorFindingsLambdaRole` Arn you created before Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html
    * In the `Properties` section create a `Action` property and specify `lambda:InvokeFunction`
    * In the `Properties` section create a `Principal` property and specify `sns.amazonaws.com`
    * In the `Properties` section create a `SourceArn` property and specify a reference to the `ContinuousAssessmentCompleteTopic` using !Ref intrinsic function

        
    <details><summary> **ARE YOU STUCK ? :(** - It's OK CLICK HERE to see the solution</summary>
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


2. **Configure SNS Topic Lambda Function Subscriber**.

    Open your notepad / text editor, edit the file named  `GoldenAMIContinuousAssesment.yml`.

    ---

    **IMPORTANT NOTE:**
    In the following steps you will be constructing your CloudFormation template in YML format.
    YML format allows you to put comments in the template by placing in # in front of the line, so it's quite handy.
    On the flip side however, it is indent sensitive, so make sure you specify the Key and values at the right level of the indentation.

    Practice makes perfect, therefore when building CloudFormation template in this lab, we will be providing a high level instruction on how to construct the template, along with the reference guide in the public documentation. The intent is so that you could get used to exploring the public documentation and get acustomed with the syntax.

    Having said that, if you are completely stuck, don't hesitate to get help from Lab instructor or, take a peek at the **SOLUTION** section if you have to.

    ---

    To find information about the properties of the resource refer to this doc: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-sns-subscription.html

    * Right under the previous resource, still inside the `Resources:` section do the following.
    * Create a resource named `ContinuousAssessmentCompleteTopicSubscription` of type `AWS::SNS::Subscription`.
    * In the `Properties` section create a `Endpoint` property and using the !GetAtt intrinsic function reference the `AnalyzeInspectorFindingsLambdaFunction` Arn you created before. Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html
    * In the `Properties` section create a `Protocol` property and specify `lambda`.
    * In the `Properties` section create a `SourceArn` property and specify a reference to the `ContinuousAssessmentCompleteTopic` using !Ref intrinsic function.

        
    <details><summary> **ARE YOU STUCK ? :(** - It's OK CLICK HERE to see the solution</summary>

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

3.  **Deploys the CloudFormation Template**

    Now that you've construct the template, it's time to update the stack, do do that please follow the [Update a Stack's Template (Console)](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-get-template.html#using-cfn-updating-stacks-get-stack.CON) guide to create your stack.

    Specify Stack `GoldenAMIContinuousAssesment` as the stack name for simplicity.

    Once you've launched your stack review the `Resources` Tab of the launch stack to identify the resouce it's created.

  Hey Hey ! Well done ! We have created all of the main resources for our solution. on top of that you've packaged this inside CloudFormation.
  Next time you need to deploy this stack, it will just take you minutes.. 

  Let's move to next page to test.