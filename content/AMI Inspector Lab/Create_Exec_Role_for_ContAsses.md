---
title: "4. Create Lambda Execution role."
weight: 4
draft: false
---

**Creating Lambda Execution Role for StartContinuousAssessmentLambdaFunction**

![](/AMI Inspector Lab/images/StartContinuousAssesmentLambdaRole.png)

1. Open your notepad / text editor, create a file named `GoldenAMIContinuousAssesment.yml`.
   Reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-iam-role.html

2. Create a resource named `StartContinuousAssessmentLambdaRole` of type `AWS::IAM::Role`.

3. In the `Properties` section add `ManagedPolicyArns` below to allow the lambda function to do basic execution and have access to Amazon Inspector APIs.
    * `arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole`
    * `arn:aws:iam::aws:policy/AmazonInspectorFullAccess`

4. In the `Properties` secion add an `AssumeRolePolicyDocument` to allow `lambda.amazonaws.com` service principal role to do an action called `sts:AssumeRole`.

5. In the `Properties` section create a policy under `Policies` to allow access to below APIs        
    * `ssm:GetParameter`
    * `ec2:DescribeImages`
    * `ec2:RunInstances`
    * `ec2:CreateTags`

<details><summary>CLICK HERE ( to see the solution ).</summary>

```
  StartContinuousAssessmentLambdaRole: 
    Properties: 
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


**Creating Lambda Execution Role for AnalyzeInspectorFindingsLambdaFunction**

![](/AMI Inspector Lab/images/AnalyzeInspectorFindingsLambdaRole.png)

1. Open your notepad / text editor, create a file named `GoldenAMIContinuousAssesment.yml`
    Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-iam-role.html

2. Create a resource named `AnalyzeInspectorFindingsLambdaRole` of type `AWS::IAM::Role`.
3. In the `Properties` section add `ManagedPolicyArns` below to allow the lambda function to do basic execution and have access to Amazon Inspector APIs 
    * `arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole`
4. In the `Properties` secion add an `AssumeRolePolicyDocument` to allow `lambda.amazonaws.com` service principal role to do an action called `sts:AssumeRole`
5. In the `Properties` section create a policy under `Policies` to allow access to below APIs        
    * `sns:Publish`
    * `ec2:DescribeInstances`
    * `ec2:TerminateInstances`
    * `inspector:AddAttributesToFindings`
    * `inspector:DescribeFindings`
    * `inspector:ListFindings`

<details><summary>CLICK HERE ( to see the solution ).</summary>
```
  AnalyzeInspectorFindingsLambdaRole: 
    Properties: 
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
