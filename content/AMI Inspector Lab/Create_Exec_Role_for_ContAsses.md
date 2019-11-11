---
title: "4. Create IAM roles for Lambda"
weight: 4
draft: false
---

In AWS, security is the highest priority. By working directly with CloudFormation, you will discover layers of security constructs that you can customize to secure your resources. One of these is the **IAM role**. For cross service interactions, AWS will leverage security services to create isolation and security boundaries for the actions executed. 

In this part of the Lab we will be defining an [IAM role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) that we will attach to our [AWS Lambda](https://aws.amazon.com/lambda/) function to specify permissions for the Lambda function. This means that the Lambda function can access only the services and resources that we will allow it to.

Not only that, we will specify these permissions using our CloudFormation template. This ensures that when we deploy out function using the AWS CloudFormation service, the security permissions are consistent.

![](/AMI Inspector Lab/images/ContinuousAssesmentLambdaRole_Full.png)

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

### Create an IAM Role for the StartContinuousAssessment function execution

In this step we will be **creating an IAM role resource** with the following criteria :
    
* The IAM Role can be _assumed_ by the AWS Lambda function
* It contains a policy that is needed for Lambda Function basic execution
* It contains a policy that is needed for full access to the Amazon Inspector service
* It contains a policy that allows it to access the AWS Systems Manager parameter store
* It contains a policy that allows it to describe and run EC2 instances, and create tags

You can read more about the IAM role resource in the CloudFormation documentation here: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-iam-role.html

Choose how you would like to build the resource from the options below:

<details><summary> **Option 1 - Build the resource manually with step-by-step instructions.**</summary>
<p>    

  * Open your text editor, create a file named `GoldenAMIContinuousAssesment.yml`.
  * Create a `Resource:` template section [Reference](https://docs.aws.amazon.com/en_pv/AWSCloudFormation/latest/UserGuide/template-anatomy.html) .
  * Create a resource named `StartContinuousAssessmentLambdaRole` of type `AWS::IAM::Role`.
  * In the `Properties` section add `ManagedPolicyArns` and list the following Arns to allow the lambda function to do basic execution and have access to Amazon Inspector APIs:
    * `arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole`
    * `arn:aws:iam::aws:policy/AmazonInspectorFullAccess`
  * In the `Properties` section add an `AssumeRolePolicyDocument` to allow the `lambda.amazonaws.com` service principal role to perform the `sts:AssumeRole` action.
  * In the `Properties` section create a policy under `Policies` to allow access to the following APIs:        
    * `ssm:GetParameter`
    * `ec2:DescribeImages`
    * `ec2:RunInstances`
    * `ec2:CreateTags`
  * In the `Properties` section, create `RoleName` and specify `StartContinuousAssessmentRole` as its value.

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


### Create a Lambda Execution Role for AnalyzeInspectorFindings

In this step we will be **creating a IAM role resource** with the criteria below:
    
* The IAM Role can be assumed by AWS the Lambda Function
* It contains a policy that is needed for Lambda Function basic execution
* It contains a policy that allows it to publish to an SNS topic
* It contains a policy that allows it to describe and terminate an EC2 instance
* It contains a policy that allows it to describe and list findings, and add attributes to findings on the Amazon Inspector service
      
You can refer here for properties reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-iam-role.html

Choose how you would like to build the resource from the options below:

<details><summary> **Option 1 - Build the resource manually with step-by-step instructions.**</summary>
<p>    

  * Open your text editor, and edit the file named `GoldenAMIContinuousAssesment.yml`.
  * Right under the previous resource, still inside the `Resources:` section, follow the next steps:
  * Create a resource named `AnalyzeInspectorFindingsLambdaRole` of type `AWS::IAM::Role`.
  * In the `Properties` section add to `ManagedPolicyArns` the following Arn to allow the lambda function to do basic execution: 
    * `arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole`
  * In the `Properties` section add an `AssumeRolePolicyDocument` to allow the `lambda.amazonaws.com` service principal role to perform the `sts:AssumeRole` action.
  * In the `Properties` section, create a policy under `Policies` to allow access to the following APIs:        
    * `sns:Publish`
    * `ec2:DescribeInstances`
    * `ec2:TerminateInstances`
    * `inspector:AddAttributesToFindings`
    * `inspector:DescribeFindings`
    * `inspector:ListFindings`
    * In the `Properties` section, create `RoleName` and specify `AnalyzeInspectorFindingsRole` as its value.

</p>
</details>

<details><summary>**Option 2 - CloudFormation template snippet**</summary>

**NOTE** The snippet below must be specified within the `Resources:` section of the CloudFormation template.


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

### Deploy the CloudFormation template

Now that you've constructed the template, it's time to deploy the stack. Please follow the [Creating a Stack on the AWS CloudFormation Console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) guide to create your stack.

Specify `GoldenAMIContinuousAssesment` as the stack name.

Once you've launched your stack, review the `Resources` tab of the launched stack to identify the resources it's created. You should see an entry with the Logical ID and the Physical ID of the IAM roles. You can click on it to go to the IAM console to see the resources created.

**Well Done!!!** 
You have packaged your IAM role in a repeatable fashion. 
Think about the next time you have to build a role for your next customer's environment. Do you still want to do it manually through the console? Hopefully not... Lets move on!