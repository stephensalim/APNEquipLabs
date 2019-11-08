---
title: "3. Create Golden AMI metadata"
weight: 3
draft: false
---

Now that we have tagged our Golden AMI for context, the next thing we need to do is to create a metadata containing information about our Golden AMI, so that our solution knows which GoldenAMI they should regularly inspect.

In this part of the lab, we will **construct a metadata containing information of our Golden AMI metadata and store it in [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)**. The metadata will be specified in JSON format and will contain th **Ami-Id** of the Golden AMI id that we tagged in the previous step, the **InstanceType** of the temporary EC2 instance we will launch to use as medium for our inspection, the **UserData** instructions, used to bootstrap Amazon Inspector agent to the temporary EC2 instance, so that Amazon Inspector can run the assesment towards the EC2.

On top of that, we will also construct the AWS Systems Manager parameter store along with the metadata entries, using [AWS CloudFormation](https://aws.amazon.com/cloudformation/) template. This will allow us to be able to rapidly deploy / remove / update these resources, the next time we need to.

![](/AMI Inspector Lab/images/StoreMetaParam.png)

### Constructing the User Data Commands

---
**IMPORTANT NOTE:**  

You must use a compatible InstanceType that is supported by your AMI, to find the compatible `InstanceType` for your golden AMI follow below steps. 

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/console/home) and navigate to the [EC2 console](https://console.aws.amazon.com/ec2/).
2.  Choose **Launch Instance**. On the **Choose an Amazon Machine Image** (**AMI**) page, choose **My AMIs**.
3.  Type the **AMI ID** that you noted in Step A in the **Search my AMIs** box, and then choose **Enter**.
4.  The search result will contain your golden AMI. To choose it, choose **Select**.
5.  Locate any available **Instance Type** and then note the corresponding value in the **Type** column.
6.  Choose **Cancel**. 

Alternatively you can use t2.large as it is monst commonly supported across all AMIs.

---

The `UserData` section, will contain [User Data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) scripts to install and start the [Amazon Inspector agent](https://docs.aws.amazon.com/inspector/latest/userguide/inspector_agents.html).
The script automates the installation of software packages when an EC2 instance launches for the first time. 

In the following steps, we will construct an operating system specific, JSON-compatible user-data script that installs and starts the Amazon Inspector agent.

1.  **Identify the command that installs the Amazon Inspector agent**

    As described in [Installing Amazon Inspector Agents](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_installing-uninstalling-agents.html) documentation, the following shell command installs the Amazon Inspector agent on an Amazon Linux-based EC2 instance. This commands essentially downloads a bash installation bashscript from AWS public repository and run a bash install command to execute the script
    ```
    wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install
    bash install
    ```
    To find this command for other operating systems, see [Installing Amazon Inspector Agents](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_installing-uninstalling-agents.html).


2.  **Identify the command that starts the Amazon Inspector agent**

    Also described in [Installing Amazon Inspector Agents](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_installing-uninstalling-agents.html) documentation, below command starts the Amazon Inspector agent on an Amazon Linux-based EC2 instance.
    ```
    sudo /etc/init.d/awsagent start
    ```
    To find this command for other operating systems, see [Amazon Inspector Agents](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_agents.html).

3.  **Create a script by concatenating the commands from the preceding two steps**

    Combining all the commands, you could consturct a script to install and start the [Amazon Inspector Agents](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_agents.html). 

    And to make this script User Data compatible you will need to explicitly specify the shell at the beginning of the script, so as a result, your script should look like below.

    ```
    #!/bin/bash
    wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install
    bash install
    sudo /etc/init.d/awsagent start
    ```

4.  **Make the user-data script JSON compatible**

    Because we need to specify the user data in JSON, and specifying new line character in JSON is a big hassle, to make it simple we can  replace all the new line characters to `;`. You can also use third party online tools to escape all the " signs and new line characters. Once you are done, your user data script shoulw look like below. 
    **(For simplicity, just use the script below in your JSON)**

    ```
    "#!/bin/bash \n wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install;bash install;/etc/init.d/awsagent start"
    ```
    
5.  **Create a JSON document of metadata of all your golden AMIs.**

    We are now ready to consturct our JSON metadata, open your notepad or text editor, copy and paste the JSON below and replace 
    * **instance-type-of-first-AMI** with the instance type you will use to launch the AMI to be inspected.
    * **AMI-ID-of-first-AMI** with the AMI id of your golden AMI.
    * **JSON-compatible-user-data-of-first-AMI** with the string in previous step above.

    Use the following template to create a JSON document:
    
    ```
    [	
        { 
            "instanceType": "instance-type-of-first-AMI", 
            "ami-id": "AMI-ID-of-first-AMI", 
            "userData": "JSON-compatible-user-data-of-first-AMI"
         }
    ]
    ```

    Once it is done, your JSON should look like this.

    ```
    [	
        { 
            "instanceType": "t2.large", 
            "ami-id": "ami-133213", 
            "userData": "#!/bin/bash \n wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install;bash install;/etc/init.d/awsagent start"
         }
    ]
    ```

    **Note:**
    If you have multiple Golden AMIs you would like to inspect simply add another entry into the list as per below example.

    ```
    [	
        { 
            "instanceType": "t2.large", 
            "ami-id": "ami-1111111111", 
            "userData": "#!/bin/bash \n wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install;bash install;/etc/init.d/awsagent start"
        },	
        { 
            "instanceType": "t2.large", 
            "ami-id": "ami-xxxxxxxxxx", 
            "userData": "#!/bin/bash \n wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install;bash install;/etc/init.d/awsagent start"
         }
    ]
    ```

6.  Once you have everything setup, your entire JSON needs to be "stringified". This is because we will need to encapsulate this JSON once again inside a string value of our CloudFormation template/

    This process is a matter of escaping the " signs in your JSON with \". You can use use online tools like https://onlinetexttools.com/json-stringify-text to do this or manually do it by hand.

    For Simplicity you can just copy the string below and **replace** the `ami-0fe2fc7acf5400b25` with your **own ami id**.

    ```
    "[{ \"instanceType\": \"t2.large\",\"ami-id\": \"ami-0fe2fc7acf5400b25\", \"userData\": \"#!/bin/bash \\n wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install;bash install;/etc/init.d/awsagent start\" }]"
    ```

    Now that you have created the Stringified JSON of your golden AMI metadata, we will store this JSON document in a Systems Manager parameter. 
    
    But instead of specifying the parameter store through the AWS console, we will be defining the resource in [AWS CloudFormation](https://aws.amazon.com/cloudformation/)

### Create an AWS Systems Manager parameter using CloudFormation

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

In this step we will be **creating a AWS Systems Manager Parameter rosurce that will contain the stringified JSON we created in the above steps. To find out how to construct this template**, you can refer to below Reference: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ssm-parameter.html. 

You can choose how you would like to construct the template from options below:

<details><summary> **Option 1 - Build the resouce manually with step by step instructions.**</summary>
<p>    

1. Open your notepad / text editor, create a file named `GoldenAMIParameters.yml`
2. Create a `Resource:` template section [Reference](https://docs.aws.amazon.com/en_pv/AWSCloudFormation/latest/UserGuide/template-anatomy.html) 
3. Create a resource under the section named `AMIMetadata` of type `AWS::SSM::Parameter`.
4. The name properties of the parameter store is to be called `ContinuousAssessmentInput`
5. The type properties of the parameter store is `String`
6. The value properties of the parameter store is the String you've created in step 6 above.
7. The Description properties is `Continuous golden AMI vulnerability assessment process metadata.`

</p>
</details>

<details><summary>  **Option 2 - CloudFormation template snippet**</summary>
<p>

```
    Resources:
        GoldenAMIParameter:
            Type: "AWS::SSM::Parameter"
            Properties:
                Name: "ContinuousAssessmentInput"
                Type: "String"
                Value: "[{ \"instanceType\": \"t2.large\",\"ami-id\": \"ami-0e2b940b603bf07f3\", \"userData\": \"#!/bin/bash \\n wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install;bash install;/etc/init.d/awsagent start\" }]"
                Description: "Continuous golden AMI vulnerability assessment process metadata."
```

Copy and paste this into the `GoldenAMIParameters.yml`, ensure to replace the ami-0e2b940b603bf07f3 with your own GoldenAMI id.

</p>
</detail>

### Deploys the CloudFormation Template

Now that you've construct the template, it's time to deploy the stack, do do that please follow the [Creating a Stack on the AWS CloudFormation Console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) guide to create your stack.

Specify Stack `GoldenAMIParameters` as the stack name for simplicity.

Once you've launched your stack review the `Resources` Tab of the launch stack to identify the resouce it's created. You should see an entry with  GoldenAMIParameter in Logical ID and ContinuousAssessmentInput in Physical ID.

[Click here](https://ap-southeast-2.console.aws.amazon.com/systems-manager/parameters?region=ap-southeast-2) to go to the Systems Manager Parameter Store console, and confirm you see the ContinuousAssesmentInput.

**Congratulations !!!**

You have built a template from scratch ! Not Bad !
But we are just about to get started, let's move on to next stage to implement the actual solution stack.