---
title: "7. Connect Analyzer Lambda to Topic"
weight: 7
draft: false
---

![](/AMI Inspector Lab/images/ContinuousAssesmentLambdaFunction04.png)

**Create Lambda Permission to allow SNS service to invoke.**.

1. Open your notepad the file you created in step 1 `GoldenAMIContinuousAssesment.yml`

2. Create a resource named `LambdaInvokePermission` of type `AWS::Lambda::Permission`.

3. In the `Properties` section create a `FunctionName` property and using the !GetAtt intrinsic function reference the `AnalyzeInspectorFindingsLambdaRole` Arn you created before
Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html

4. In the `Properties` section create a `Action` property and specify `lambda:InvokeFunction`

5. In the `Properties` section create a `Principal` property and specify `sns.amazonaws.com`

6. In the `Properties` section create a `SourceArn` property and specify a reference to the `ContinuousAssessmentCompleteTopic` using !Ref intrinsic function

        
<details><summary>CLICK HERE ( to see the solution ).</summary>
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

![](/AMI Inspector Lab/images/ContinuousAssesmentLambdaFunction05.png)

**Configure SNS Topic Lambda Function Subscriber**.


1. Open your notepad the file you created in step 1 `GoldenAMIContinuousAssesment.yml`

2. Create a resource named `LambdaInvokePermission` of type `AWS::SNS::Subscription`.

3. In the `Properties` section create a `Endpoint` property and using the !GetAtt intrinsic function reference the `AnalyzeInspectorFindingsLambdaFunction` Arn you created before
Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html

4. In the `Properties` section create a `Protocol` property and specify `lambda`

5. In the `Properties` section create a `SourceArn` property and specify a reference to the `ContinuousAssessmentCompleteTopic` using !Ref intrinsic function

        
<details><summary>CLICK HERE ( to see the solution ).</summary>
```
  ContinuousAssessmentCompleteTopicSubscription: 
    Properties: 
      Endpoint: !GetAtt "AnalyzeInspectorFindingsLambdaFunction.Arn"
      Protocol: lambda
      TopicArn: !Ref "ContinuousAssessmentCompleteTopic"
    Type: "AWS::SNS::Subscription"
```
</details>