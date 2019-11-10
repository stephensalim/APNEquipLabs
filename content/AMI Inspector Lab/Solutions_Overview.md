---
title: "1. Solution Overview"
weight: 1
draft: false
---

![](/AMI Inspector Lab/images/Main.png)

---

**IMPORTANT NOTE:** 

For this solution to work, you must deploy all the resources in the AWS Region where you build your golden AMIs. 
The region must have [Amazon Inspector](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_supported_os_regions.html#inspector_supported-regions) available. 
For the best lab experience, please choose the Sydney Region **(ap-southeast-2)**.

---

Here is an overview of how the solutions works:

1.  A scheduled [CloudWatch Event](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html) will trigger the [AWS Lambda](http://aws.amazon.com/lambda/) function called `StartContinuousAssessment` every morning at 6 AM. This essentially acts like a cron job (on Linux) or a scheduled task (on Windows) that will run on a regular basis.

2.  The `StartContinuousAssessment` [AWS Lambda](http://aws.amazon.com/lambda/) function will first gather information about which Golden AMI it should asses. This is done by downloading the metadata information about the Golden AMI that we store inside [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html). AWS Systems Manager Parameter Store provides secure, hierarchical storage for configuration data and secrets management. 

3.  For each of the Golden AMIs specified in the metadata, the `StartContinuousAssessment` function will launch an EC2 instance using that specific AMI.

4.  The EC2 instances will then use [EC2 User Data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) to bootstrap the installation and start up of the [Amazon Inspector](https://aws.amazon.com/inspector/) agent. This agent is needed for the Amazon Inspector Service to inspect the EC2 AMI image at the operating system level.

5.  The `StartContinuousAssessment` function will trigger [Amazon Inspector's](https://aws.amazon.com/inspector/) inspection of the EC2 instance's security posture according to [CIS](https://docs.aws.amazon.com/inspector/latest/userguide/inspector_cis.html) and [CVE](https://docs.aws.amazon.com/inspector/latest/userguide/inspector_cves.html).
        
6.  Once [Amazon Inspector](https://aws.amazon.com/inspector/) completes the inspection, a notification will be sent to an [SNS Topic](https://docs.aws.amazon.com/sns/latest/dg/welcome.html) which will then forward a message to the email subscribers to the topic, as well as triggering the `AnalyzeInspectorFindings` [AWS Lambda](http://aws.amazon.com/lambda/) function where the next activity takes place.

7. The `AnalyzeInspectorFindings` function will tag [Amazon Inspector's](https://aws.amazon.com/inspector/) result with relevant information from the AMI & EC2 Instance for context. It will then also clean up the temporary EC2 instance that was launched in order to be used by Amazon Inspector to analyze the Golden AMI's security posture.

Now that the steps are clear, lets start building! On to the next page! **[`^0^]/** 

---

**NOTE:**

In order to provide a reference and to give context on what you will be building, the **final state of the CloudFormation template** can be found below. Feel free to check your progress against this template, or, test the outcome by copying the template source into a YAML text file and create the stack.

<details><summary>**[Expand to view the template]**</summary>
<p>    


```
Parameters: 
  Email: 
    Type: String
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
  LambdaInvokePermission: 
    Properties: 
      Action: "lambda:InvokeFunction"
      FunctionName: !GetAtt "AnalyzeInspectorFindingsLambdaFunction.Arn"
      Principal: sns.amazonaws.com
      SourceArn: !Ref "ContinuousAssessmentCompleteTopic"
    Type: "AWS::Lambda::Permission"
  ContinuousAssessmentCompleteTopicSubscription: 
    Properties: 
      Endpoint: !GetAtt "AnalyzeInspectorFindingsLambdaFunction.Arn"
      Protocol: lambda
      TopicArn: !Ref "ContinuousAssessmentCompleteTopic"
    Type: "AWS::SNS::Subscription"
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
                    template = inspector.create_assessment_template(assessmentTargetArn=target['assessmentTargetArn'],assessmentTemplateName='ContinuousAssessment', durationInSeconds=300,rulesPackageArns=rules['rulesPackageArns'])
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
      FunctionName: "StartContinuousAssessment"
    Type: "AWS::Lambda::Function"
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
                print(event)
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
      FunctionName: "AnalyzeInspectorFindings"
    Type: "AWS::Lambda::Function"
  ContinuousGoldenAMIAssessmentTrigger:
    Type: AWS::Events::Rule
    Properties:
      Name: ContinuousGoldenAMIAssessmentTrigger
      ScheduleExpression: "cron(0 6 * * ? *)"
      State: ENABLED
      Targets:
        -
          Arn:
            Fn::GetAtt:
              - "StartContinuousAssessmentLambdaFunction"
              - "Arn"
          Input: '{"AMIsParamName": "ContinuousAssessmentInput"}'
          Id: ContinuousGoldenAMIAssessmentTrigger
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
  ContinuousAssessmentResultsTopic: 
    Type: "AWS::SNS::Topic"
    Properties:
      TopicName: ContinuousAssessmentResultsTopic
      Subscription:
        - Endpoint: !Ref Email
          Protocol: "email"
```
<p>
</details>


---