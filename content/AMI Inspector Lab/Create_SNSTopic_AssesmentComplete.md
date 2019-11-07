---
title: "5. Create SNS Topic "
weight: 5
draft: false
---

Now that we have our IAM role defined in our Stack, we shall move on to our next resources.
In this solutin we will be levaraging a number of [SNS](https://docs.aws.amazon.com/sns/latest/dg/welcome.html) Topics. These topics will be used to notify our users about the state if the Golden AMI inspection, as well as act as a decoupling structure to connect our `StartContinousAssesment` function and `AnalyzeInspectorFindings` Function.

![](/AMI Inspector Lab/images/Role&ContinuousAsssesmentSNSTopic_Full.png)

Let's follow below steps.

1. **Creating the SNS Topic for StartContinousAssesment Completion**

    Open your notepad / text editor, edit the file named  `GoldenAMIContinuousAssesment.yml`.

    ---

    **IMPORTANT NOTE:**
    In the following steps you will be constructing your CloudFormation template in YML format.
    YML format allows you to put comments in the template by placing in # in front of the line, so it's quite handy.
    On the flip side however, it is indent sensitive, so make sure you specify the Key and values at the right level of the indentation.

    Practice makes perfect, therefore when building CloudFormation template in this lab, we will be providing a high level instruction on how to construct the template, along with the reference guide in the public documentation. The intent is so that you could get used to exploring the public documentation and get acustomed with the syntax.

    Having said that, if you are completely stuck, don't hesitate to get help from Lab instructor or, take a peek at the **SOLUTION** section if you have to.

    ---

    To find information about the properties of the resource refer to this doc: 
    https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sns-topic.html
    https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sns-policy.html

    * Right under the previous resource, still inside the `Resources:` section do the following.
    * Create a resource named `ContinuousAssessmentCompleteTopic` of type `AWS::SNS::Topic`.
    * No need to put in any `Properties` in this resource, keep all empty / default.
    * Create another resource named `ContinuousAssessmentCompleteTopicPolicy` of type `AWS::SNS::TopicPolicy`.  
    * In the `Properties` section create a `PolicyDocument` and allow service `inspector.amazonaws.com` to do action `sns:Publish` on all resource.
      Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sns-policy.html
    * In the `Properties` section create a `Topics` and Reference the SNS Topic we just created in this step using !Ref intrinsic function.
      Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html

    Notice that we are creating SNSTopic policy and allowing AWS Inspector service to publish message to the topic. This is an additional security construct that you can take advantage on to allow isolation between services.

    <details><summary> **ARE YOU STUCK ? :(** - It's OK CLICK HERE to see the solution</summary>

    **READ >>** Below snippet must be specified within `Resources:` section of the cloudformation template

    ```
      ContinuousAssessmentCompleteTopic: 
        Type: "AWS::SNS::Topic"
      ContinuousAssessmentCompleteTopicPolicy: 
        Properties: 
          PolicyDocument: 
            Id: MyTopicPolicy
            Statement: 
              - 
                Action: "sns:Publish"
                Effect: Allow
                Principal: 
                  Service: inspector.amazonaws.com
                Resource: "*"
                Sid: My-statement-id
            Version: "2012-10-17"
          Topics: 
            - !Ref "ContinuousAssessmentCompleteTopic"
        Type: "AWS::SNS::TopicPolicy"
    ```
    </details>

2. **Creating the SNS Topic for AnalyzeInspectorFindings Result.**

    Open your notepad / text editor, edit the file named  `GoldenAMIContinuousAssesment.yml`.

    ---

    **IMPORTANT NOTE:**
    In the following steps you will be constructing your CloudFormation template in YML format.
    YML format allows you to put comments in the template by placing in # in front of the line, so it's quite handy.
    On the flip side however, it is indent sensitive, so make sure you specify the Key and values at the right level of the indentation.

    Practice makes perfect, therefore when building CloudFormation template in this lab, we will be providing a high level instruction on how to construct the template, along with the reference guide in the public documentation. The intent is so that you could get used to exploring the public documentation and get acustomed with the syntax.

    Having said that, if you are completely stuck, don't hesitate to get help from Lab instructor or, take a peek at the **SOLUTION** section if you have to.

    ---

    To find information about the properties of the resource refer to this doc: 
    https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sns-topic.html

    * Right under the previous resource, still inside the `Resources:` section do the following.
    * Create a resource named `ContinuousAssessmentResultsTopic` of type `AWS::SNS::Topic`.
    * No need to put in any `Properties` in this resource for now, keep all empty / default.

    <details><summary> **ARE YOU STUCK ? :(** - It's OK CLICK HERE to see the solution</summary>

    **READ >>** Below snippet must be specified within `Resources:` section of the cloudformation template.

    ```
      ContinuousAssessmentResultsTopic: 
        Type: "AWS::SNS::Topic"
    ```
    </details>

3.  **Deploys the CloudFormation Template**

    Now that you've construct the template, it's time to update the stack, do do that please follow the [Update a Stack's Template (Console)](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-get-template.html#using-cfn-updating-stacks-get-stack.CON) guide to create your stack.

    Specify Stack `GoldenAMIContinuousAssesment` as the stack name for simplicity.

    Once you've launched your stack review the `Resources` Tab of the launch stack to identify the resouce it's created. You should see an entry with in Logical ID and in Physical ID of the SNS Topic.

Let's now move to creating our Lambda Function.