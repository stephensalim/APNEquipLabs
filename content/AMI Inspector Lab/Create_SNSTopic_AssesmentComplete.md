---
title: "5. Create SNS Topic "
weight: 5
draft: false
---

Now that we have our IAM role defined in our stack, we shall move on to the next resources.
In this solution we will be using a number of [SNS Topics](https://docs.aws.amazon.com/sns/latest/dg/welcome.html). These topics will be used to notify our users about the state of the Golden AMI inspection. They will also be used to decouple the `StartContinousAssesment` and `AnalyzeInspectorFindings` AWS Lambda functions. With an SNS Topic IAM Policy, you can define which AWS service is allowed to publish to the SNS topic.

![](/AMI Inspector Lab/images/Role&ContinuousAsssesmentSNSTopic_Full.png)

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

### Creating the SNS Topic for StartContinousAssesment completion

In this step we will be **creating SNS an Topic and an IAM Policy** as follows:
    
* The SNS Topic name is `ContinuousAssessmentCompleteTopic`
* The topic has an email Subscriber, with an email address that you can provide dynamically as a template parameter
* The SNS Topic policy allows `inspector.amazonaws.com` to send messages to SNS Topic

Refer to the relevant CloudFormation references here: 

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sns-topic.html
https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sns-policy.html

Choose how you would like to build the resources from the options below:

<details><summary> **Option 1 - Build the resources manually with step-by-step instructions.**</summary>
<p>    
      
  * Open your text editor, and edit the file named `GoldenAMIContinuousAssesment.yml`.
  * Create a `Parameters` section, and under it create a new parameter named `Email` with type `String`.
  * Inside the `Resources:` section add the following resources:
  * Create a resource named `ContinuousAssessmentCompleteTopic` of type `AWS::SNS::Topic`.
  * In the `Properties` section create a `TopicName` with `ContinuousAssessmentCompleteTopic` as its value.
  * In the `Properties` section create a `Subscription` and create an item with a key named `Protocol` and `email` as it's value.
  * Under the same Item, create a new key called `Endpoint` and reference the `Email` Parameter you crated earlier in the step using `!Ref` intrinsic function. [Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html)
  * Create another resource named `ContinuousAssessmentCompleteTopicPolicy` of type `AWS::SNS::TopicPolicy`.  
  * In the `Properties` section create a `PolicyDocument` and allow service `inspector.amazonaws.com` to perform the `sns:Publish` action on all resources. [Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sns-policy.html)
  * In the `Properties` section, add `Topics` and reference the SNS Topic we just created in this step using `!Ref` intrinsic function. [Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html)

  Notice that we are creating SNS Topic Policy and allowing AWS Inspector service to publish message to the topic. This is an additional security measure that you can take advantage of to enforce isolation between services.

</p>
</details>


<details><summary>**Option 2 - CloudFormation template snippet**</summary>

    
```
    Parameters: 
      Email: 
        Type: String
```

**NOTE** The snippet below must be specified the `Resources:` section of the CloudFormation template.

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



### Creating the SNS Topic for AnalyzeInspectorFindings results
    
In this step we will be **creating an SNS Topic** as follows:
    
* The SNS Topic Name is `ContinuousAssessmentResultsTopic`
* The topic has an email Subscriber, with an email address that you can provide dynamically as a template parameter

Refer to the relevant CloudFormation reference here: 
https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sns-topic.html

Choose how you would like to build the resource from the options below:

<details><summary> **Option 1 - Build the resource manually with step-by-step instructions.**</summary>
<p>  

  * Open your text editor, and edit the file named  `GoldenAMIContinuousAssesment.yml`.
  * Add the following resources to the `Resources:` section:
  * Create a resource named `ContinuousAssessmentResultsTopic` of type `AWS::SNS::Topic`.
  * In the `Properties` section create a `TopicName` with a value of `ContinuousAssessmentResultsTopic`.
  * In the `Properties` section create a `Subscription` and create a item with a key named `Protocol` with a value of `email`.
  * In the same item, create a new key called `Endpoint` and reference the `Email` Parameter you crated earlier in the previous step using the `!Ref` intrinsic function. [Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html)

</p>
</details>

<details><summary>**Option 2 - CloudFormation template snippet**</summary>

**NOTE** The snippet below must be specified within the `Resources:` section of the CloudFormation template.

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

### Deploy the CloudFormation template

Now that you've updated the template, it's time to update the stack as well. 
    
* To do that please follow the [Update a Stack's Template (Console)](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-get-template.html#using-cfn-updating-stacks-get-stack.CON) guide.
* Specify Stack `GoldenAMIContinuousAssesment` as the stack name.
* Specify your email address as the Email parameter in the wizard.
    
Once you've updated your stack:

* Review the `Resources` Tab of the stack to identify the new resources that have been created. 
* You should see entries for the SNS Topics you have created.
* Check your email address you specified as the stack parameter - you should see 2 email subscription confirmation requests from the 2 SNS Topics. 
* Click on the link to confirm the subscriptions.

Let's now move on to creating our Lambda function.