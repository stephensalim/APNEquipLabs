---
title: "5. Create SNS Topic "
weight: 5
draft: false
---

**Creating the SNS Topic for Assesment completion**

![](/AMI Inspector Lab/images/ContinuousAsssesmentCompleteTopicSNS.png)

**Create Inspection Complete SNS Topic & Policy to allow access for AW Inspector**.

1. Open your notepad the file you created in step 1 `GoldenAMIContinuousAssesment.yml`.

2. Create a resource named `ContinuousAssessmentCompleteTopic` of type `AWS::SNS::Topic`.

3. No need to put in any `Properties` in this resource, keep all empty / default.

4. Create another resource named `ContinuousAssessmentCompleteTopicPolicy` of type `AWS::SNS::TopicPolicy`.  

5. In the `Properties` section create a `PolicyDocument` and allow service `inspector.amazonaws.com` to do action `sns:Publish` on all resource.

    Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sns-policy.html

6. In the `Properties` section create a `Topics` and Reference the SNS Topic we just created in this step using !Ref intrinsic function.

    Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html

<details><summary>CLICK HERE ( to see the solution ).</summary>
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
