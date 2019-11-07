---
title: "4. Create Lambda Execution role"
weight: 4
draft: false
---

In AWS Security is at highest priority, through working directly with CloudFormation, you will discover layers of security construct that you could customize to secure your resource. One of it is IAM role, for cross service interactions AWS will levarage security services to create isolation and security boundaries on the service executed. 

In this part of the Lab we will be defining an [IAM role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) that we will attach to our [AWS Lambda](https://aws.amazon.com/lambda/) function and specify permissions for the Lambda function to access only the services and resource we allow it to.

Not only that, we will also specify this inside our CloudFormation template, so we are ensured that when we deploy these function using our CloudFormation, the security permission is consistent.

![](/AMI Inspector Lab/images/ContinuousAssesmentLambdaRole_Full.png)

1. **Creating Lambda Execution Role for StartContinuousAssessment Function**

    Open your notepad / text editor, create a file named `GoldenAMIContinuousAssesment.yml`.

    ---

    **IMPORTANT NOTE:**
    In the following steps you will be constructing your CloudFormation template in YML format.
    YML format allows you to put comments in the template by placing in # in front of the line, so it's quite handy.
    On the flip side however, it is indent sensitive, so make sure you specify the Key and values at the right level of the indentation.

    Practice makes perfect, therefore when building CloudFormation template in this lab, we will be providing a high level instruction on how to construct the template, along with the reference guide in the public documentation. The intent is so that you could get used to exploring the public documentation and get acustomed with the syntax.

    Having said that, if you are completely stuck, don't hesitate to get help from Lab instructor or, take a peek at the **SOLUTION** section if you have to.

    ---

    To find information about the properties of the resource refer to this doc: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-iam-role.html
    
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

    <details><summary> **ARE YOU STUCK ? :(** - It's OK CLICK HERE to see the solution</summary>

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


2. **Creating Lambda Execution Role for AnalyzeInspectorFindings**

    Open your notepad / text editor, edit the file named `GoldenAMIContinuousAssesment.yml`

    ---

    **IMPORTANT NOTE:**
    In the following steps you will be constructing your CloudFormation template in YML format.
    YML format allows you to put comments in the template by placing in # in front of the line, so it's quite handy.
    On the flip side however, it is indent sensitive, so make sure you specify the Key and values at the right level of the indentation.

    Practice makes perfect, therefore when building CloudFormation template in this lab, we will be providing a high level instruction on how to construct the template, along with the reference guide in the public documentation. The intent is so that you could get used to exploring the public documentation and get acustomed with the syntax.

    Having said that, if you are completely stuck, don't hesitate to get help from Lab instructor or, take a peek at the **SOLUTION** section if you have to.

    ---

    To find information about the properties of the resource refer to this doc: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-iam-role.html

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

    <details><summary> **ARE YOU STUCK ? :(** - It's OK CLICK HERE to see the solution</summary>

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

8.  **Deploys the CloudFormation Template**

    Now that you've construct the template, it's time to deploy the stack, do do that please follow the [Creating a Stack on the AWS CloudFormation Console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) guide to create your stack.

    Specify Stack `GoldenAMIContinuousAssesment` as the stack name for simplicity.

    Once you've launched your stack review the `Resources` Tab of the launch stack to identify the resouce it's created. You should see an entry with in Logical ID and in Physical ID. of the IAM role. You can click on it to go to the IAM console to see the resources created.

**Well Done !** you have packaged your IAM role in a repeatable fashion, think about the next time you have to build this on your next customer environment. Do you still want to build it manually through the console ? Hopefully not. Let us move on !