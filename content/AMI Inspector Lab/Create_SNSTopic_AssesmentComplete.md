---
title: "5. Create SNS Topic "
weight: 5
draft: false
---

Now that we have our IAM role defined in our Stack, we shall move on to our next resources.
In this solutin we will be levaraging a number of [SNS](https://docs.aws.amazon.com/sns/latest/dg/welcome.html) Topics. These topics will be used to notify our users about the state if the Golden AMI inspection, as well as act as a decoupling structure to connect our `StartContinousAssesment` function and `AnalyzeInspectorFindings` Function. With SNS Topic Policy, you can define which service on AWS is allowed to publish to the SNS topic.

![](/AMI Inspector Lab/images/Role&ContinuousAsssesmentSNSTopic_Full.png)

Let's follow below steps.

1. **Creating the SNS Topic for StartContinousAssesment Completion**

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

    Reference: 

    https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sns-topic.html
    https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sns-policy.html

    * Open your notepad / text editor, edit the file named  `GoldenAMIContinuousAssesment.yml`.
    * Create a `Parameters` section, and under it create a new parameter named `Email` with type `String`.
    * Right under the previous resource, still inside the `Resources:` section do the following.
    * Create a resource named `ContinuousAssessmentCompleteTopic` of type `AWS::SNS::Topic`.
    * In the `Properties` section create a `TopicName` and put in `ContinuousAssessmentCompleteTopic` as it's value.
    * In the `Properties` section create a `Subscription` and create a Item with key named `Protocol` and `email` as it's value.
    * Under the same Item, create a new key called `Endpoint` and reference the `Email` Parameters you crated earlier in the step using !Ref intrinsic function.
      Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html
    * Create another resource named `ContinuousAssessmentCompleteTopicPolicy` of type `AWS::SNS::TopicPolicy`.  
    * In the `Properties` section create a `PolicyDocument` and allow service `inspector.amazonaws.com` to do action `sns:Publish` on all resource.
      Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sns-policy.html
    * In the `Properties` section create a `Topics` and Reference the SNS Topic we just created in this step using !Ref intrinsic function.
      Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html

    Notice that we are creating SNSTopic policy and allowing AWS Inspector service to publish message to the topic. This is an additional security construct that you can take advantage on to allow isolation between services.

    <details><summary> **ARE YOU STUCK ? :(** - It's OK CLICK HERE to see the solution</summary>

    **READ >>** Below snippet must be specified within `Resources:` section of the cloudformation template
    
    ```
    Parameters: 
      Email: 
        Type: String
    ```

    ```
      ContinuousAssessmentCompleteTopic: 
        Type: "AWS::SNS::Topic"
        Properties:
          TopicName: ContinuousAssessmentCompleteTopic
          Subscription:
            - Endpoint: !Ref Email
              Protocol: "email"
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

    Reference: 
    https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sns-topic.html

    * Open your notepad / text editor, edit the file named  `GoldenAMIContinuousAssesment.yml`.
    * Right under the previous resource, still inside the `Resources:` section do the following.
    * Create a resource named `ContinuousAssessmentResultsTopic` of type `AWS::SNS::Topic`.
    * In the `Properties` section create a `TopicName` and put in `ContinuousAssessmentResultsTopic` as it's value
    * In the `Properties` section create a `Subscription` and create a Item with key named `Protocol` and `email` as it's value.
    * Under the same Item, create a new key called `Endpoint` and reference the `Email` Parameters you crated earlier in the previous step using !Ref intrinsic function.
      Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html

    <details><summary> **ARE YOU STUCK ? :(** - It's OK CLICK HERE to see the solution</summary>

    **READ >>** Below snippet must be specified within `Resources:` section of the cloudformation template.

    ```
      ContinuousAssessmentResultsTopic: 
        Type: "AWS::SNS::Topic"
        Properties:
          TopicName: ContinuousAssessmentResultsTopic
          Subscription:
            - Endpoint: !Ref Email
              Protocol: "email"
    ```
    </details>

3.  **Deploys the CloudFormation Template**

    Now that you've construct the template, it's time to update the stack. 
    
    * To do that please follow the [Update a Stack's Template (Console)](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-get-template.html#using-cfn-updating-stacks-get-stack.CON) guide to create your stack.
    * Specify Stack `GoldenAMIContinuousAssesment` as the stack name for simplicity.
    * Specify your email address as the Email parameter in the wizard.
    
    Once you've launched your stack :

    * Review the `Resources` Tab of the launch stack to identify the resouce it's created. 
    * You should see an entry with in Logical ID and in Physical ID of the SNS Topic.
    * Check your email address you specified before, you should see 2 email subscription confirmation request from the 2 SNS Topics. 
    * Click on the link to confirm them.

Let's now move to creating our Lambda Function.