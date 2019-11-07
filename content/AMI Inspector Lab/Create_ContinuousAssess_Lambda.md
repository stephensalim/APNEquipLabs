---
title: "6. Create Lambda Functions"
weight: 6
draft: false
---

Alright, now for the meaty part !
In this part of the lab, we will walk through how to construct / package [AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html) Function and attach it to the IAM roles we created in previous step. We will be creating 2 Lambda Functions `StartContinuousAssesment` &  `AnalyzeInspectorFindings` Each of this Lambda Functions will also have a connection to SNS Topics we created in previous steps through a Lambda Environment variable we defined in the function.

![](/AMI Inspector Lab/images/ContinuousAssesmentLambdaFunction02.png)


1. **Creating the Lambda Function StartContinuousAssessment**

    Open your notepad / text editor, edit the file named  `GoldenAMIContinuousAssesment.yml`.

    ---

    **IMPORTANT NOTE:**
    In the following steps you will be constructing your CloudFormation template in YML format.
    YML format allows you to put comments in the template by placing in # in front of the line, so it's quite handy.
    On the flip side however, it is indent sensitive, so make sure you specify the Key and values at the right level of the indentation.

    Practice makes perfect, therefore when building CloudFormation template in this lab, we will be providing a high level instruction on how to construct the template, along with the reference guide in the public documentation. The intent is so that you could get used to exploring the public documentation and get acustomed with the syntax.

    Having said that, if you are completely stuck, don't hesitate to get help from Lab instructor or, take a peek at the **SOLUTION** section if you have to.

    ---

    To find information about the properties of the resource refer to this doc: 
    https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-function.html

    * Right under the previous resource, still inside the `Resources:` section do the following.
    * Create a resource named `StartContinuousAssessmentLambdaFunction` of type `AWS::Lambda::Function`.
    * In the `Properties` section create a `Role` property and using the !GetAtt intrinsic function reference the IAM role Arn you created in previous step Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html
    * In the `Properties` section create a `Code` property and specify the lambda function code using in line code, copu & paste below. To place in multiple line, in yaml you can use the | sign then place the remaining code below it. Example : https://aws.amazon.com/blogs/infrastructure-and-automation/deploying-aws-lambda-functions-using-aws-cloudformation-the-portable-way/

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

    * In the `Properties` section create a `Environment` property and specify an environment variable named `AssesmentCompleteTopicArn` and reference the SNS topic resource created in previous step  To do this, you can use !Ref intrinsic function. Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html
        
<details><summary> **ARE YOU STUCK ? :(** - It's OK CLICK HERE to see the solution</summary>

**READ >>** Below snippet must be specified within `Resources:` section of the cloudformation template.

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
      FunctionName: "StartContinuousAssessment"
    Type: "AWS::Lambda::Function"
```

</details>


2. **Create Lambda Function to AnalyzeInspectorFindings**.


    Open your notepad / text editor, edit the file named  `GoldenAMIContinuousAssesment.yml`.

    ---

    **IMPORTANT NOTE:**
    In the following steps you will be constructing your CloudFormation template in YML format.
    YML format allows you to put comments in the template by placing in # in front of the line, so it's quite handy.
    On the flip side however, it is indent sensitive, so make sure you specify the Key and values at the right level of the indentation.

    Practice makes perfect, therefore when building CloudFormation template in this lab, we will be providing a high level instruction on how to construct the template, along with the reference guide in the public documentation. The intent is so that you could get used to exploring the public documentation and get acustomed with the syntax.

    Having said that, if you are completely stuck, don't hesitate to get help from Lab instructor or, take a peek at the **SOLUTION** section if you have to.

    ---

    * Right under the previous resource, still inside the `Resources:` section do the following.
    * Create a resource named `AnalyzeInspectorFindingsLambdaFunction` of type `AWS::Lambda::Function`.
    * In the `Properties` section create a `Role` property and using the !GetAtt intrinsic function reference the IAM role Arn you created for `AnalyzeInspectorFindingsLambdaRole`. Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html
    * In the `Properties` section create a `Code` property and specify the lambda function code using in line code, copu & paste below. To place in multiple line, in yaml you can use the | sign then place the remaining code below it. Example : https://aws.amazon.com/blogs/infrastructure-and-automation/deploying-aws-lambda-functions-using-aws-cloudformation-the-portable-way/

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


    * In the `Properties` section create a `Environment` property and specify an environment variable named `ContinuousAssessmentResultsTopic` and reference the SNS topic resource `ContinuousAssessmentCompleteTopic`. To do this, you can use !Ref intrinsic function. Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html
        
<details><summary> **ARE YOU STUCK ? :(** - It's OK CLICK HERE to see the solution</summary>

**READ >>** Below snippet must be specified within `Resources:` section of the cloudformation template.


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
      FunctionName: "AnalyzeInspectorFindings"
    Type: "AWS::Lambda::Function"
```
</details>

3.  **Deploys the CloudFormation Template**

    Now that you've construct the template, it's time to update the stack, do do that please follow the [Update a Stack's Template (Console)](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-get-template.html#using-cfn-updating-stacks-get-stack.CON) guide to create your stack.

    Specify Stack `GoldenAMIContinuousAssesment` as the stack name for simplicity.

    Once you've launched your stack review the `Resources` Tab of the launch stack to identify the resouce it's created. You should see an entry with in Logical ID and in Physical ID of the Lambda Functions, you can click on the corresponding link under Physical ID to go to the Lambda Functions console.