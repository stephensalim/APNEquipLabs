---
title: "6. Create Lambda Functions"
weight: 6
draft: false
---

Alright, now for the meaty part !
In this part of the lab, we will walk through how to construct / package [AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html) Function and attach it to the IAM roles we created in previous step. We will be creating 2 Lambda Functions `StartContinuousAssesment` & `AnalyzeInspectorFindings` Each of this Lambda Functions will also have a connection to SNS Topics we created in previous steps through a Lambda Environment variable we defined in the function.

![](/AMI Inspector Lab/images/ContinuousAssesmentLambdaFunction02.png)

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

### Creating the Lambda Function StartContinuousAssessment

In this step we will be **creating a Lambda Function** with below criteria :
    
* FunctionName called `StartContinuousAssessment`
* Handler to the function is `index.lambda_handler`.
* MemorySize is 512, and timeout is 5 mins.
* Create Environment variable named `AssesmentCompleteTopicArn` and reference the arn of `ContinuousAssessmentCompleteTopic` SNS Topic.
* Specify Roles, and reference the ARN of IAM role called `StartContinuousAssessmentLambdaRole`.
* Specify the code inline within the template, Code for this function is available below.

You can refer here for properties reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-function.html


**Code**

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
            template = inspector.create_assessment_template(assessmentTargetArn=target['assessmentTargetArn'],assessmentTemplateName='ContinuousAssessment', durationInSeconds=300,rulesPackageArns=rules['rulesPackageArns'])
            assessmentTemplateArn=template['assessmentTemplateArn']
            response = inspector.subscribe_to_event(event='ASSESSMENT_RUN_COMPLETED',resourceArn=template['assessmentTemplateArn'],topicArn=os.environ['AssesmentCompleteTopicArn']) 
            print('Template Created:'+template['assessmentTemplateArn'])
        else:
            assessmentTemplateArn=existingTemplates.get('assessmentTemplateArns')[0]
        time.sleep(240)
        run = inspector.start_assessment_run(assessmentTemplateArn=assessmentTemplateArn,assessmentRunName='ContinuousAssessment'+'-'+str(millis))
        return 'Done'
```



Choose one of the option below on how you want to build the resource.

<details><summary>**Option 1 - Build the resouce manually with step by step instructions.**</summary>
<p>    

* Open your notepad / text editor, edit the file named  `GoldenAMIContinuousAssesment.yml`.
* Right under the previous resource, still inside the `Resources:` section do the following.
* Create a resource named `StartContinuousAssessmentLambdaFunction` of type `AWS::Lambda::Function`.
* Under `Properties` section create `FunctionName` with `StartContinuousAssessment` as the value.
* Under `Properties` create `Handler` with `index.lambda_handler` as the value.
* Under `Properties` create `MemorySize` with `512` as the value.
* Under `Properties` create `Runtime` with `python3.6` as the value.
* Under `Properties` create `Timeout` with `300` as the value.
* Under `Properties` create `Environment` under it, following after that, create `Variables` under `Environment`.
    * Under `Variables` then create a key called `AssesmentCompleteTopicArn` and reference the value `ContinuousAssessmentCompleteTopic` using !Ref intrinsic function. Reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html
* In the `Properties` section create a `Role` property and using the !GetAtt intrinsic function reference the IAM role Arn you created in previous step called `StartContinuousAssessmentLambdaRole` Reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html
* In the `Properties` section create a `Code` property and specify the lambda function code using in line code, copu & paste below. To place in multiple line, in yaml you can use the | sign then place the remaining code below it. Example : https://aws.amazon.com/blogs/infrastructure-and-automation/deploying-aws-lambda-functions-using-aws-cloudformation-the-portable-way/

</p>
</details>

<details><summary>**Option 2 - CloudFormation template snippet**</summary>

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
```

</details>


### Create Lambda Function to AnalyzeInspectorFindings


In this step we will be **creating a Lambda Function** with below criteria :
    
* FunctionName called `AnalyzeInspectorFindings`
* Handler to the function is `index.lambda_handler`.
* MemorySize is 512, and timeout is 5 mins.
* Create Environment variable named `ContinuousAssessmentResultsTopic` and reference the arn of `ContinuousAssessmentResultsTopic` SNS Topic.
* Specify Roles, and reference the ARN of IAM role called `AnalyzeInspectorFindingsLambdaRole`.
* Specify the code inline within the template, Code for this function is available below.

You can refer here for properties reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-function.html


**Code** 

```
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
```

Choose one of the option below on how you want to build the resource.

<details><summary>**Option 1 - Build the resouce manually with step by step instructions.**</summary>
<p>    

* Open your notepad / text editor, edit the file named  `GoldenAMIContinuousAssesment.yml`.
* Right under the previous resource, still inside the `Resources:` section do the following.
* Create a resource named `AnalyzeInspectorFindingsLambdaFunction` of type `AWS::Lambda::Function`.
* Under `Properties` section create `FunctionName` with `AnalyzeInspectorFindings` as the value.
* Under `Properties` create `Handler` with `index.lambda_handler` as the value.
* Under `Properties` create `MemorySize` with `512` as the value.
* Under `Properties` create `Runtime` with `python3.6` as the value.
* Under `Properties` create `Timeout` with `300` as the value.
* Under `Properties` create `Environment` under it, following after that, create `Variables` under `Environment`.
    * Under `Variables` then create a key called `ContinuousAssessmentResultsTopic` and reference the value `ContinuousAssessmentResultsTopic` using !Ref intrinsic function. Reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html
* In the `Properties` section create a `Role` property and using the !GetAtt intrinsic function reference the IAM role Arn you created in previous step called `AnalyzeInspectorFindingsLambdaRole` Reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html
* In the `Properties` section create a `Code` property and specify the lambda function code using in line code, copu & paste below. To place in multiple line, in yaml you can use the | sign then place the remaining code below it. Example : https://aws.amazon.com/blogs/infrastructure-and-automation/deploying-aws-lambda-functions-using-aws-cloudformation-the-portable-way/

</p>
</details>

 
<details><summary>**Option 2 - CloudFormation template snippet**</summary>

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
```
</details>

### Deploys the CloudFormation Template

Now that you've construct the template, it's time to update the stack, do do that please follow the [Update a Stack's Template (Console)](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-get-template.html#using-cfn-updating-stacks-get-stack.CON) guide to create your stack.

Specify Stack `GoldenAMIContinuousAssesment` as the stack name for simplicity.

Once you've launched your stack review the `Resources` Tab of the launch stack to identify the resouce it's created. You should see an entry with in Logical ID and in Physical ID of the Lambda Functions, you can click on the corresponding link under Physical ID to go to the Lambda Functions console.