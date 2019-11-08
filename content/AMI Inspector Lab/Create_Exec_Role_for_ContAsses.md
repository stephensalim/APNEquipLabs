---
title: "4. Create IAM roles for Lambda"
weight: 4
draft: false
---

In AWS Security is at highest priority, through working directly with CloudFormation, you will discover layers of security construct that you could customize to secure your resource. One of it is IAM role, for cross service interactions AWS will levarage security services to create isolation and security boundaries on the service executed. 

In this part of the Lab we will be defining an [IAM role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) that we will attach to our [AWS Lambda](https://aws.amazon.com/lambda/) function and specify permissions for the Lambda function to access only the services and resource we allow it to.

Not only that, we will also specify this inside our CloudFormation template, so we are ensured that when we deploy these function using our CloudFormation, the security permission is consistent.

![](/AMI Inspector Lab/images/ContinuousAssesmentLambdaRole_Full.png)

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

### Creating IAM Role for StartContinuousAssessment Function execution

In this step we will be **creating a IAM role resource** with below criteria :
    
* IAM Role can be assumed by AWS Lambda Function
* Contains policy that is needed for Lambda Function Basic executions
* Contains policy that is needed for Full access to Amazon Inspector service
* Contains policy that allows you to Get Systems Manager parameter store
* Contains policy that allows you to describe, run instance, and create tags.

You can refer here for properties reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-iam-role.html

Choose one of the option below on how you want to build the resource.
 
<details><summary>**Option 1 - Build the resouce manually with step by step instructions.**</summary>
<p>    

  * Open your notepad / text editor, create a file named `GoldenAMIContinuousAssesment.yml`.
  * Create a `Resource:` template section [Reference](https://docs.aws.amazon.com/en_pv/AWSCloudFormation/latest/UserGuide/template-anatomy.html) 
  * Create a resource named `StartContinuousAssessmentLambdaRole` of type `AWS::IAM::Role`.
  * In the `Properties` section add `ManagedPolicyArns` below to allow the lambda function to do basic execution and have access to Amazon Inspector APIs.
    * `arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole`
    * `arn:aws:iam::aws:policy/AmazonInspectorFullAccess`
  * In the `Properties` secion add an `AssumeRolePolicyDocument` to allow `lambda.amazonaws.com` service principal role to do an action called `sts:AssumeRole`.
  * In the `Properties` section create a policy under `Policies` to allow access to below APIs        
    * `ssm:GetParameter`
    * `ec2:DescribeImages`
    * `ec2:RunInstances`
    * `ec2:CreateTags`
  * In the `Properties` section create `RoleName` and specify `StartContinuousAssessmentRole` as it's value

</p>
</details>

<details><summary>**Option 2 - CloudFormation template snippet**</summary>

```
    Resources:
      StartContinuousAssessmentLambdaRole: 
        Properties:
          RoleName: "StartContinuousAssessmentRole"
          AssumeRolePolicyDocument: 
            Statement: 
              - 
                Action: 
                  - "sts:AssumeRole"
                Effect: Allow
                Principal: 
                  Service: 
                    - lambda.amazonaws.com
            Version: "2012-10-17"
          ManagedPolicyArns: 
            - "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
            - "arn:aws:iam::aws:policy/AmazonInspectorFullAccess"
          Path: /
          Policies: 
            - 
              PolicyDocument: 
                Statement: 
                  - 
                    Action: 
                      - "ssm:GetParameter"
                      - "ec2:DescribeImages"
                      - "ec2:RunInstances"
                      - "ec2:CreateTags"
                    Effect: Allow
                    Resource: "*"
                    Sid: StartContinuousAssessmentLambdaPolicyStmt
                Version: "2012-10-17"
              PolicyName: root
        Type: "AWS::IAM::Role"
```
</details>


### Creating Lambda Execution Role for AnalyzeInspectorFindings

In this step we will be **creating a IAM role resource** with below criteria :
    
* IAM Role can be assumed by AWS Lambda Function
* Contains policy that is needed for Lambda Function Basic executions
* Contains policy that allows you to publish sns topic
* Contains policy that allows you to describe, terminate instance
* Contains policy that allows you to describe findings, list findings, add attributes to findings on amazon inspector.
      
You can refer here for properties reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-iam-role.html

Choose one of the option below on how you want to build the resource.
 
<details><summary>**Option 1 - Build the resouce manually with step by step instructions.**</summary>
<p>    

  * Open your notepad / text editor, edit the file named `GoldenAMIContinuousAssesment.yml`
  * Right under the previous resource, still inside the `Resources:` section do the following.
  * Create a resource named `AnalyzeInspectorFindingsLambdaRole` of type `AWS::IAM::Role`.
  * In the `Properties` section add `ManagedPolicyArns` below to allow the lambda function to do basic execution and have access to Amazon Inspector APIs 
    * `arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole`
  * In the `Properties` secion add an `AssumeRolePolicyDocument` to allow `lambda.amazonaws.com` service principal role to do an action called `sts:AssumeRole`
  * In the `Properties` section create a policy under `Policies` to allow access to below APIs        
    * `sns:Publish`
    * `ec2:DescribeInstances`
    * `ec2:TerminateInstances`
    * `inspector:AddAttributesToFindings`
    * `inspector:DescribeFindings`
    * `inspector:ListFindings`
    * In the `Properties` section create `RoleName` and specify `AnalyzeInspectorFindingsRole` as it's value

</p>
</details>

<details><summary>**Option 2 - CloudFormation template snippet**</summary>

**READ >>** Below snippet must be specified within `Resources:` section of the cloudformation template.


```
      AnalyzeInspectorFindingsLambdaRole: 
        Properties:
          RoleName: "AnalyzeInspectorFindingsRole"
          AssumeRolePolicyDocument: 
            Statement: 
              - 
                Action: 
                  - "sts:AssumeRole"
                Effect: Allow
                Principal: 
                  Service: 
                    - lambda.amazonaws.com
            Version: "2012-10-17"
          ManagedPolicyArns: 
            - "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
          Path: /
          Policies: 
            - 
              PolicyDocument: 
                Statement: 
                  - 
                    Action: 
                      - "sns:Publish"
                      - "ec2:DescribeInstances"
                      - "ec2:TerminateInstances"
                      - "inspector:AddAttributesToFindings"
                      - "inspector:DescribeFindings"
                      - "inspector:ListFindings"
                    Effect: Allow
                    Resource: "*"
                    Sid: AnalyzeInspectorFindingsLambdaPolicyStmt
                Version: "2012-10-17"
              PolicyName: AnalyzeInspectorFindingsLambdaPolicy
        Type: "AWS::IAM::Role"
```
</details>

### Deploys the CloudFormation Template

Now that you've construct the template, it's time to deploy the stack, do do that please follow the [Creating a Stack on the AWS CloudFormation Console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) guide to create your stack.

Specify Stack `GoldenAMIContinuousAssesment` as the stack name for simplicity.

Once you've launched your stack review the `Resources` Tab of the launch stack to identify the resouce it's created. You should see an entry with in Logical ID and in Physical ID. of the IAM role. You can click on it to go to the IAM console to see the resources created.

**Well Done !** you have packaged your IAM role in a repeatable fashion, think about the next time you have to build this on your next customer environment. Do you still want to build it manually through the console ? Hopefully not. Let us move on !