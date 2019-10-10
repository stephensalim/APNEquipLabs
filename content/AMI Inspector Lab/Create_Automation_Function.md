---
title: "4. Create Automation Functions"
weight: 4
draft: false
---

In this section we will create, create a couple of lambda functions to execute the AMI inspection as well as notify users of the findings


**Creating Lambda Execution Role for StartContinuousAssessmentLambdaFunction**

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

**Creating the SNS Topic for Assesment completion**

1. Open your notepad the file you created in step 1 `GoldenAMIContinuousAssesment.yml`

2. Create a resource named `ContinuousAssessmentCompleteTopic` of type `AWS::SNS::Topic`.

3. No need to put in any `Properties`, keep all empty / default 

<details><summary>CLICK HERE ( to see the solution ).</summary>
```
  ContinuousAssessmentCompleteTopic: 
    Type: "AWS::SNS::Topic"
```
</details>

**Creating the Lambda Fucntion to Inspect the AMI**

1. Open your notepad the file you created in step 1 `GoldenAMIContinuousAssesment.yml`

2. Create a resource named `StartContinuousAssessmentLambdaFunction` of type `AWS::Lambda::Function`.

3. In the `Properties` section create a `Role` property and using the !GetAtt intrinsic function reference the IAM role Arn you created in step 1.
Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html

4. In the `Properties` section create a `Code` property and specify the lambda function code using in line code instead of specifying the s3 bucket location paste in below.
To place in multiple line, in yaml you can use the | sign then place the remaining code below it.

    Example : https://aws.amazon.com/blogs/infrastructure-and-automation/deploying-aws-lambda-functions-using-aws-cloudformation-the-portable-way/

    Code 
    ```
    import json
    import urllib.parse
    import boto3
    import time
    import os
    def lambda_handler(event, context):
        AMIsParamName = event['AMIsParamName'];
        region=os.environ['AWS_DEFAULT_REGION']
        ec2 = boto3.client('ec2',region)
        ssm = boto3.client('ssm',region)
        inspector = boto3.client('inspector',region)
        AmiJson =  ssm.get_parameter(Name=AMIsParamName)['Parameter']['Value']
        print(AmiJson)
        items = json.loads(AmiJson)
        for entry in items:
            images= ec2.describe_images(ImageIds=[entry['ami-id']],DryRun=False)
            tags = images['Images'][0]['Tags']
            tags.append({'Key': 'continuous-assessment-instance', 'Value': 'true'})
            ec2.run_instances(ImageId=entry['ami-id'],InstanceType=entry['instanceType'],UserData=entry['userData'],DryRun=False,MaxCount=1,MinCount=1,TagSpecifications=[{'ResourceType': 'instance','Tags': tags}])
        assessmentTemplateArn='';
        rules = inspector.list_rules_packages();
        
        millis = int(round(time.time() * 1000))
        existingTemplates = inspector.list_assessment_templates(filter={'namePattern': 'ContinuousAssessment'})
        print('Total templates found:'+str(len(existingTemplates['assessmentTemplateArns'])))
        if len(existingTemplates['assessmentTemplateArns'])==0:
            resGroup = inspector.create_resource_group(resourceGroupTags=[{'key': 'continuous-assessment-instance','value': 'true'}])
            target = inspector.create_assessment_target(assessmentTargetName='ContinuousAssessment',resourceGroupArn=resGroup['resourceGroupArn'])
            template = inspector.create_assessment_template(assessmentTargetArn=target['assessmentTargetArn'],assessmentTemplateName='ContinuousAssessment', durationInSeconds=3600,rulesPackageArns=rules['rulesPackageArns'])
            assessmentTemplateArn=template['assessmentTemplateArn']
            response = inspector.subscribe_to_event(event='ASSESSMENT_RUN_COMPLETED',resourceArn=template['assessmentTemplateArn'],topicArn=os.environ['AssesmentCompleteTopicArn']) 
            print('Template Created:'+template['assessmentTemplateArn'])
        else:
            assessmentTemplateArn=existingTemplates.get('assessmentTemplateArns')[0]
        time.sleep(240)
        run = inspector.start_assessment_run(assessmentTemplateArn=assessmentTemplateArn,assessmentRunName='ContinuousAssessment'+'-'+str(millis))
        return 'Done'
    ```

5. In the `Properties` section create a `Environment` property and specify an environment variable named `AssesmentCompleteTopicArn` and reference the SNS topic resource created in step 2. 
To do this, you can use !Ref intrinsic function.

    Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html
        
<details><summary>CLICK HERE ( to see the solution ).</summary>
```
  StartContinuousAssessmentLambdaFunction: 
    Properties: 
      Code: 
        ZipFile: |
            import json
            import urllib.parse
            import boto3
            import time
            import os
            def lambda_handler(event, context):
                AMIsParamName = event['AMIsParamName'];
                region=os.environ['AWS_DEFAULT_REGION']
                ec2 = boto3.client('ec2',region)
                ssm = boto3.client('ssm',region)
                inspector = boto3.client('inspector',region)
                AmiJson =  ssm.get_parameter(Name=AMIsParamName)['Parameter']['Value']
                print(AmiJson)
                items = json.loads(AmiJson)
                for entry in items:
                    images= ec2.describe_images(ImageIds=[entry['ami-id']],DryRun=False)
                    tags = images['Images'][0]['Tags']
                    tags.append({'Key': 'continuous-assessment-instance', 'Value': 'true'})
                    ec2.run_instances(ImageId=entry['ami-id'],InstanceType=entry['instanceType'],UserData=entry['userData'],DryRun=False,MaxCount=1,MinCount=1,TagSpecifications=[{'ResourceType': 'instance','Tags': tags}])
                assessmentTemplateArn='';
                rules = inspector.list_rules_packages();
                
                millis = int(round(time.time() * 1000))
                existingTemplates = inspector.list_assessment_templates(filter={'namePattern': 'ContinuousAssessment'})
                print('Total templates found:'+str(len(existingTemplates['assessmentTemplateArns'])))
                if len(existingTemplates['assessmentTemplateArns'])==0:
                    resGroup = inspector.create_resource_group(resourceGroupTags=[{'key': 'continuous-assessment-instance','value': 'true'}])
                    target = inspector.create_assessment_target(assessmentTargetName='ContinuousAssessment',resourceGroupArn=resGroup['resourceGroupArn'])
                    template = inspector.create_assessment_template(assessmentTargetArn=target['assessmentTargetArn'],assessmentTemplateName='ContinuousAssessment', durationInSeconds=3600,rulesPackageArns=rules['rulesPackageArns'])
                    assessmentTemplateArn=template['assessmentTemplateArn']
                    response = inspector.subscribe_to_event(event='ASSESSMENT_RUN_COMPLETED',resourceArn=template['assessmentTemplateArn'],topicArn=os.environ['AssesmentCompleteTopicArn']) 
                    print('Template Created:'+template['assessmentTemplateArn'])
                else:
                    assessmentTemplateArn=existingTemplates.get('assessmentTemplateArns')[0]
                time.sleep(240)
                run = inspector.start_assessment_run(assessmentTemplateArn=assessmentTemplateArn,assessmentRunName='ContinuousAssessment'+'-'+str(millis))
                return 'Done'
      Environment: 
        Variables: 
          AssesmentCompleteTopicArn: !Ref "ContinuousAssessmentCompleteTopic"
      Handler: index.lambda_handler
      MemorySize: 512
      Role: !GetAtt "StartContinuousAssessmentLambdaRole.Arn"
      Runtime: python3.6
      Timeout: 300
    Type: "AWS::Lambda::Function"
```
</details>

Once this is set up, the next thing to do is to set up the SNS Topic that AWS Inspector will trigger upon completion of the inspection. Below steps to create the resources.

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

**Creating Lambda Execution Role for AnalyzeInspectorFindingsLambdaFunction**

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


**Create Lambda Function to AnalyzeInspectorFindings**.

1. Open your notepad the file you created in step 1 `GoldenAMIContinuousAssesment.yml`

2. Create a resource named `AnalyzeInspectorFindingsLambdaFunction` of type `AWS::Lambda::Function`.

3. In the `Properties` section create a `Role` property and using the !GetAtt intrinsic function reference the IAM role Arn you created for `AnalyzeInspectorFindingsLambdaRole`.
Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html

4. In the `Properties` section create a `Code` property and specify the lambda function code using in line code instead of specifying the s3 bucket location paste in below.
To place in multiple line, in yaml you can use the | sign then place the remaining code below it.

    Example : https://aws.amazon.com/blogs/infrastructure-and-automation/deploying-aws-lambda-functions-using-aws-cloudformation-the-portable-way/

    Code 
```
            import json 
            import os
            import boto3
            import collections
            import ast
            def lambda_handler(event, context): 
                message = event['Records'][0]['Sns']['Message'] 
                jsonVal = json.loads(message);
                assessmentArn =jsonVal['run']  
                region=os.environ['AWS_DEFAULT_REGION']
                ec2 = boto3.client('ec2',region) 
                sns = boto3.client('sns',region) 
                inspector = boto3.client('inspector',region) 
                findingArns = inspector.list_findings(assessmentRunArns=[jsonVal['run']],maxResults=5000)
                aggregateData={}
                for findingArn in findingArns['findingArns']:
                    finding = inspector.describe_findings(findingArns=[findingArn]) 
                    for result in finding['findings']: 
                        instanceId =result['assetAttributes']['agentId']
                        severity =result['severity']
                        cveName=result['id']
                        if not (instanceId) in aggregateData:
                            aggregateData[instanceId]={}
                            aggregateData[instanceId]['findings']={}
                            aggregateData[instanceId]['findings'][severity]=0
                            instance=ec2.describe_instances(InstanceIds=[instanceId]);
                            tagsStr=str(instance['Reservations'][0]['Instances'][0]['Tags']) 
                            tagsStr =tagsStr.replace('Key','key').replace('Value','value')  
                            aggregateData[instanceId]['tags']= ast.literal_eval(tagsStr)
                        elif not (severity) in aggregateData[instanceId]['findings']:
                            aggregateData[instanceId]['findings'][severity]=0
                        aggregateData[instanceId]['findings'][severity]=aggregateData[instanceId]['findings'][severity]+1; 
                        inspector.add_attributes_to_findings(findingArns=[result['arn']],attributes=aggregateData[instanceId]['tags'])
                        print(findingArns=[result['arn']])
                        print(attributes=aggregateData[instanceId]['tags'])
                tagsList=[]
                for key  in aggregateData: 
                    outputJson=[] 
                    for tag in aggregateData[key]['tags']:
                        if tag['key'] != 'continuous-assessment-instance':
                            outputJson.append("\""+tag['key']+"\""+":"+"\""+tag['value']+"\"")
                    for sev in aggregateData[key]['findings']:
                        outputJson.append("\"Finding-Severity-"+sev+"-Count\""+":"+"\""+str(aggregateData[key]['findings'][sev])+"\"")
                    outputJson.sort()
                    print(outputJson)
                    tagsList.append('{'+', '.join(outputJson)+'}')
                    print('Terminating:'+key)
                    ec2.terminate_instances(InstanceIds=[key],DryRun=False)
                sns.publish(TopicArn=os.environ['ContinuousAssessmentResultsTopic'],Message='['+', '.join(tagsList)+']')
                return jsonVal['run']
```


5. In the `Properties` section create a `Environment` property and specify an environment variable named `ContinuousAssessmentResultsTopic` and reference the SNS topic resource `ContinuousAssessmentCompleteTopic` 
To do this, you can use !Ref intrinsic function.

    Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html
        
<details><summary>CLICK HERE ( to see the solution ).</summary>
```
  AnalyzeInspectorFindingsLambdaFunction: 
    Properties: 
      Code: 
        ZipFile: |
            import json 
            import os
            import boto3
            import collections
            import ast
            def lambda_handler(event, context): 
                message = event['Records'][0]['Sns']['Message'] 
                jsonVal = json.loads(message);
                assessmentArn =jsonVal['run']  
                region=os.environ['AWS_DEFAULT_REGION']
                ec2 = boto3.client('ec2',region) 
                sns = boto3.client('sns',region) 
                inspector = boto3.client('inspector',region) 
                findingArns = inspector.list_findings(assessmentRunArns=[jsonVal['run']],maxResults=5000)
                aggregateData={}
                for findingArn in findingArns['findingArns']:
                    finding = inspector.describe_findings(findingArns=[findingArn]) 
                    for result in finding['findings']: 
                        instanceId =result['assetAttributes']['agentId']
                        severity =result['severity']
                        cveName=result['id']
                        if not (instanceId) in aggregateData:
                            aggregateData[instanceId]={}
                            aggregateData[instanceId]['findings']={}
                            aggregateData[instanceId]['findings'][severity]=0
                            instance=ec2.describe_instances(InstanceIds=[instanceId]);
                            tagsStr=str(instance['Reservations'][0]['Instances'][0]['Tags']) 
                            tagsStr =tagsStr.replace('Key','key').replace('Value','value')  
                            aggregateData[instanceId]['tags']= ast.literal_eval(tagsStr)
                        elif not (severity) in aggregateData[instanceId]['findings']:
                            aggregateData[instanceId]['findings'][severity]=0
                        aggregateData[instanceId]['findings'][severity]=aggregateData[instanceId]['findings'][severity]+1; 
                        inspector.add_attributes_to_findings(findingArns=[result['arn']],attributes=aggregateData[instanceId]['tags'])
                        print(findingArns=[result['arn']])
                        print(attributes=aggregateData[instanceId]['tags'])
                tagsList=[]
                for key  in aggregateData: 
                    outputJson=[] 
                    for tag in aggregateData[key]['tags']:
                        if tag['key'] != 'continuous-assessment-instance':
                            outputJson.append("\""+tag['key']+"\""+":"+"\""+tag['value']+"\"")
                    for sev in aggregateData[key]['findings']:
                        outputJson.append("\"Finding-Severity-"+sev+"-Count\""+":"+"\""+str(aggregateData[key]['findings'][sev])+"\"")
                    outputJson.sort()
                    print(outputJson)
                    tagsList.append('{'+', '.join(outputJson)+'}')
                    print('Terminating:'+key)
                    ec2.terminate_instances(InstanceIds=[key],DryRun=False)
                sns.publish(TopicArn=os.environ['ContinuousAssessmentResultsTopic'],Message='['+', '.join(tagsList)+']')
                return jsonVal['run']
      Environment: 
        Variables: 
          ContinuousAssessmentResultsTopic: !Ref "ContinuousAssessmentResultsTopic"
      Handler: index.lambda_handler
      MemorySize: 512
      Role: !GetAtt "AnalyzeInspectorFindingsLambdaRole.Arn"
      Runtime: python3.6
      Timeout: 300
    Type: "AWS::Lambda::Function"
```
</details>


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