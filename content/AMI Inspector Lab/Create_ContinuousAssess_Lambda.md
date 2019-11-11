---
title: "6. Create Lambda Functions"
weight: 6
draft: false
---

All right, now for the exciting part!
In this section of the lab, we will walk through how to construct and package [AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html) Functions and attach them to the IAM roles we created in the previous step. We will be creating 2 Lambda Functions `StartContinuousAssesment` and `AnalyzeInspectorFindings`. We will be providing references to the SNS Topics you created in the previous steps to each of these Lambda functions through an environment variable.

![](/AMI Inspector Lab/images/ContinuousAssesmentLambdaFunction02.png)

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

### Create the StartContinuousAssessment Lambda function 

In this step we will be **creating a Lambda Function** as follows:
    
* The FunctionName is `StartContinuousAssessment`
* The function Handler is `index.lambda_handler`.
* The MemorySize is 512, and Timeout is 5 minutes.
* Create an Environment Variable named `AssesmentCompleteTopicArn` and reference the Arn of the `ContinuousAssessmentCompleteTopic` SNS Topic.
* The function Roles must reference the Arn of IAM role called `StartContinuousAssessmentLambdaRole`.
* Specify the code inline within the template. Code for this function is available below.

You can refer here for the properties reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-function.html


**Code**

```
    import json
    import urllib.parse
    import boto3
    import time
    import os
    def lambda_handler(event, context):
        AMIsParamName = event['AMIsParamName'];
        region = os.environ['AWS_DEFAULT_REGION']
        ec2 = boto3.client('ec2',region)
        ssm = boto3.client('ssm',region)
        inspector = boto3.client('inspector',region)
        AmiJson =  ssm.get_parameter(Name=AMIsParamName)['Parameter']['Value']
        print(AmiJson)
        items = json.loads(AmiJson)
        for entry in items:
            images = ec2.describe_images(ImageIds=[entry['ami-id']],DryRun=False)
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



Choose how you would like to build the resource from the options below:

<details><summary> **Option 1 - Build the resource manually with step-by-step instructions.**</summary>
<p>    

* Open your text editor, and edit the file named  `GoldenAMIContinuousAssesment.yml`.
* Add the following resource to the `Resources:` section:
* Create a resource named `StartContinuousAssessmentLambdaFunction` of type `AWS::Lambda::Function`.
* Under `Properties` add `FunctionName` with `StartContinuousAssessment` as the value.
* Under `Properties` add `Handler` with `index.lambda_handler` as the value.
* Under `Properties` add `MemorySize` with `512` as the value.
* Under `Properties` add `Runtime` with `python3.6` as the value.
* Under `Properties` add `Timeout` with `300` as the value.
* Under `Properties` add `Environment`, and nested under it, create `Variables`.
    * Under `Variables`create a key called `AssesmentCompleteTopicArn` and reference the value `ContinuousAssessmentCompleteTopic` using the `!Ref` intrinsic function. [Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html)
* Back in the `Properties` section, create a `Role` property and using the `!GetAtt` intrinsic function, reference the IAM role Arn you created in the previous step called `StartContinuousAssessmentLambdaRole`. [Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html)
* In the `Properties` section create a `Code` property and specify the Lambda function source code inline. Copy the function code provided. To specify multiple lines for a YAML value, you can use the `|` character to add in a multi-line value after the key name. [Example](https://aws.amazon.com/blogs/infrastructure-and-automation/deploying-aws-lambda-functions-using-aws-cloudformation-the-portable-way/)

</p>
</details>

<details><summary>**Option 2 - CloudFormation template snippet**</summary>

**NOTE** The snippet below must be specified within the `Resources:` section of the CloudFormation template.

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
                region = os.environ['AWS_DEFAULT_REGION']
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


### Create the AnalyzeInspectorFindings Lambda function 


In this step we will be **creating a Lambda Function** as follows :
    
* The FunctionName is `AnalyzeInspectorFindings`
* The Handler of the function is `index.lambda_handler`.
* The MemorySize is 512, and Timeout is 5 mins.
* Create an Environment variable named `ContinuousAssessmentResultsTopic` and reference the Arn of the `ContinuousAssessmentResultsTopic` SNS Topic.
* The function Role must reference the Arn of the IAM role called `AnalyzeInspectorFindingsLambdaRole`.
* Specify the code inline within the template. Code for this function is available below.

You can refer here for the properties reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-function.html


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
                    assessmentArn = jsonVal['run']  
                    region = os.environ['AWS_DEFAULT_REGION']
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

Choose how you would like to build the resource from the options below:

<details><summary> **Option 1 - Build the resource manually with step-by-step instructions.**</summary>
<p>    

* Open your text editor, and edit the file named  `GoldenAMIContinuousAssesment.yml`.
* Add the following to the `Resources:` section:
* Create a resource named `AnalyzeInspectorFindingsLambdaFunction` of type `AWS::Lambda::Function`.
* Under `Properties` add `FunctionName` with `AnalyzeInspectorFindings` as the value.
* Under `Properties` add `Handler` with `index.lambda_handler` as the value.
* Under `Properties` add `MemorySize` with `512` as the value.
* Under `Properties` add `Runtime` with `python3.6` as the value.
* Under `Properties` add `Timeout` with `300` as the value.
* Under `Properties` add `Environment`, and nested under it, create `Variables`.
    * Under `Variables`create a key called `ContinuousAssessmentResultsTopic` and reference the value `ContinuousAssessmentResultsTopic` using the `!Ref` intrinsic function. [Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html)
* Back in the `Properties` section, create a `Role` property and using the `!GetAtt` intrinsic function, reference the IAM role Arn you created in the previous step called `ContinuousAssessmentResultsTopic`. [Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html)
* In the `Properties` section create a `Code` property and specify the Lambda function source code inline. Copy the function code provided. To specify multiple lines for a YAML value, you can use the `|` character to add in a multi-line value after the key name. [Example](https://aws.amazon.com/blogs/infrastructure-and-automation/deploying-aws-lambda-functions-using-aws-cloudformation-the-portable-way/)

</p>
</details>

 
<details><summary>**Option 2 - CloudFormation template snippet**</summary>

**NOTE** The snippet below must be specified within the `Resources:` section of the CloudFormation template.


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
                assessmentArn = jsonVal['run']  
                region = os.environ['AWS_DEFAULT_REGION']
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



### Deploy the CloudFormation template

Now that you've updated the template, it's time to update the stack as well. 
    
To do that please follow the [Update a Stack's Template (Console)](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-get-template.html#using-cfn-updating-stacks-get-stack.CON) guide.

Once you've updated your stack review the `Resources` Tab of the stack to identify the new resources that have been created for you. You should see an entries for the Lambda functions, you can click on the corresponding link under Physical ID to go to the Lambda console.
