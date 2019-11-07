---
title: "3. Store Golden AMI metadata."
weight: 3
draft: false
---

 ![](/AMI Inspector Lab/images/StoreMetaParam.png)

In this section we will walk through the process of construction the golden AMI metadata and store them in [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html) our metadata will be specified in JSON format and will contain the following information for each golden AMI:

* The **Ami-Id** is the golden AMI id that we tagged in the previous step ( Hopefully you've taken note of that, otherwise please review the previous step ).
* The **InstanceType** is the EC2 instance type our solution will use to launch the golden AMI with and run the inspection on.
* The **UserData** section will contain our bootstrap instructions to install AWS Inspector Agent.

---
**IMPORTANT NOTE:**  You must use a compatible InstanceType that is supported by your AMI, to find the compatible `InstanceType` for your golden AMI follow below steps. (But for simplicity you can use **t2.large** this has been tested). 

Each AMI has a list of compatible `InstanceTypes`. The `InstanceType` is a required parameter for launching an EC2 instance from a golden AMI. To find a compatible `InstanceType` for your golden AMI:

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/console/home) and navigate to the [EC2 console](https://console.aws.amazon.com/ec2/).
2.  Choose **Launch Instance**. On the **Choose an Amazon Machine Image** (**AMI**) page, choose **My AMIs**.
3.  Type the **AMI ID** that you noted in Step A in the **Search my AMIs** box, and then choose **Enter**.
4.  The search result will contain your golden AMI. To choose it, choose **Select**.
5.  Locate any available **Instance Type** and then note the corresponding value in the **Type** column.
6.  Choose **Cancel**. 

The solution will launch the chosen `InstanceType` every time the vulnerability assessment runs.

---

For the `UserData` section you will need to create a [User Data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) script to install and start the [Amazon Inspector agent](https://docs.aws.amazon.com/inspector/latest/userguide/inspector_agents.html).
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

    And to make this script User Data compatible you will need to explicitly specify the shell at the beginning of the script, so in practice your script should look like below

    ```
    #!/bin/bash
    wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install
    bash install
    sudo /etc/init.d/awsagent start
    ```

4.  **Make the user-data script JSON compatible**

    Now, because we need to specify the user data in JSON, and specifying new line character in JSON is a big hassle, to make it simple we can  replace all the new line characters to `;`. You can use third party online tools to escape all the " signs and new line characters.
    Once you are done, your user data script shoulw look like below. (For simplicity, just use the script below in your JSON).

    **JSON-compatible-user-data-of-first-AMI**
    ```
    "#!/bin/bash \n wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install;bash install;/etc/init.d/awsagent start"
    ```
    
5.  **Create a JSON document of metadata of all your golden AMIs.**

    We are now ready to consturct our JSON metadata, ppen your notepad or text editor copy and paste the JSON below and replace 
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

    once it is done, your JSON should look like this.

    ```
    [	
        { 
            "instanceType": "t2.large", 
            "ami-id": "ami-133213", 
            "userData": "#!/bin/bash \n wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install;bash install;/etc/init.d/awsagent start"
         }
    ]
    ```

6.  Once you have everything setup your JSON needs to be converted into a string, because we will need to encapsulate this JSON once again inside a string value of our CloudFormation template ( you will see this in next step.)

    This is just a matter of converting your JSON and escaping the " signs. You can use use online tools like https://onlinetexttools.com/json-stringify-text or manually do it by hand.

    Again for simplicity you can just copy the string below and **replace** the `ami-0fe2fc7acf5400b25` with your **prefered ami**.

    ```
    "[{ \"instanceType\": \"t2.large\",\"ami-id\": \"ami-0fe2fc7acf5400b25\", \"userData\": \"#!/bin/bash \\n wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install;bash install;/etc/init.d/awsagent start\" }]"
    ```

    Now that you have created the Stringified JSON of your golden AMI metadata, we will store this JSON document in a Systems Manager parameter. But instead of specifying the parameter store through the console, we will be using [AWS CloudFormation](https://aws.amazon.com/cloudformation/) to allow us to make this process repeatable in the future.. Why do we do this? Because we are AWSome ! Follow below steps to do so.

7.  **Create an AWS Systems Manager parameter using CloudFormation**
    
    Open your notepad / text editor, create a file named `GoldenAMIParameters.yml`
    
    ---

    **IMPORTANT NOTE:**
    In the following steps you will be constructing your CloudFormation template in YML format.
    YML format allows you to put comments in the template by placing in # in front of the line, so it's quite handy.
    On the flip side however, it is indent sensitive, so make sure you specify the Key and values at the right level of the indentation.

    Practice makes perfect, therefore when building CloudFormation template in this lab, we will be providing a high level instruction on how to construct the template, along with the reference guide in the public documentation. The intent is so that you could get used to exploring the public documentation and get acustomed with the syntax.

    Having said that, if you are completely stuck, don't hesitate to get help from Lab instructor or, take a peek at the **SOLUTION** section if you have to.

    ---

    To find information about the properties of the resource refer to this doc: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ssm-parameter.html

    * Create a `Resource:` template section [Reference](https://docs.aws.amazon.com/en_pv/AWSCloudFormation/latest/UserGuide/template-anatomy.html) 
    * Create a resource under the section named `AMIMetadata` of type `AWS::SSM::Parameter`.
    * The name properties of the parameter store is to be called `ContinuousAssessmentInput`
    * The type properties of the parameter store is `String`
    * The value properties of the parameter store is the String you've created in step 6 above.
    * The Description properties is `Continuous golden AMI vulnerability assessment process metadata.`

    <details><summary> **ARE YOU STUCK ? :(** - It's OK CLICK HERE to see the solution</summary>
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

    Copy and paste this into the `GoldenAMIParameters.yml`

    </p>
    </detail>

8.  **Deploys the CloudFormation Template**

    Now that you've construct the template, it's time to deploy the stack, do do that please follow the [Creating a Stack on the AWS CloudFormation Console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) guide to create your stack.

    Specify Stack `GoldenAMIParameters` as the stack name for simplicity.

    Once you've launched your stack review the `Resources` Tab of the launch stack to identify the resouce it's created. You should see an entry with  GoldenAMIParameter in Logical ID and ContinuousAssessmentInput in Physical ID.

    [Click here](https://ap-southeast-2.console.aws.amazon.com/systems-manager/parameters?region=ap-southeast-2) to go to the Systems Manager Parameter Store console, and confirm you see the ContinuousAssesmentInput.

**Congratulations !!!**

You have built a template from scratch ! Not Bad !
But we are just about to get started, let's move on to next stage to implement the actual solution stack.