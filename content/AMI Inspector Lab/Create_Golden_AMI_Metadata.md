---
title: "3. Create Golden AMI metadata"
weight: 3
draft: false
---

Now that we have tagged our Golden AMI for context, the next thing we need to do is to create metadata containing information about our Golden AMI, so that our solution knows which Golden AMIs they should regularly inspect.

In this part of the lab, we will **construct  metadata containing information about our Golden AMI and store it in the [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)**. The metadata will be specified in JSON format and will contain the **AMI ID** of the Golden AMI that we tagged in the previous step, the **Instance Type** of the temporary EC2 instance we will launch to use as medium for our inspection, and the **User Data** instructions needed to bootstrap Amazon Inspector agent, so that Amazon Inspector can run the security posture assessment against the EC2 instance.

We will construct the AWS Systems Manager parameter store, along with the metadata entries, using an [AWS CloudFormation](https://aws.amazon.com/cloudformation/) template. This will allow us to rapidly deploy, remove, or update these resources as required.

![](/AMI Inspector Lab/images/StoreMetaParam.png)

### Constructing the User Data Commands

---
**IMPORTANT NOTE:**  

You must use a compatible **Instance Type** that is supported by your AMI. To find this, follow the steps below: 

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/console/home) and navigate to the [EC2 console](https://console.aws.amazon.com/ec2/).
2.  Choose **Launch Instance**. On the **Choose an Amazon Machine Image** (**AMI**) page, choose **My AMIs**. _(If you used an AWS Public AMI for the lab, leave **Quick Start** selected instead.)_
3.  Type the **AMI ID** that you noted for your golden AMI in the **Search my AMIs** box, and then hit **Enter**. 
4.  The search result will contain your golden AMI. To choose it, choose **Select**.
5.  Locate any available **Instance Type** and then note the corresponding value in the **Type** column.
6.  Choose **Cancel**. 

Alternatively you can use the **t2.large** instance type, as it commonly supported across all AMIs.

---

The `UserData` section contains [user-data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) scripts to install and start the [Amazon Inspector agent](https://docs.aws.amazon.com/inspector/latest/userguide/inspector_agents.html).
The script automates the installation of software packages when an EC2 instance launches for the first time. 

In the following steps, we will construct an operating system specific, JSON-compatible, user-data script that installs and starts the Amazon Inspector agent.

1.  **Identify the command that installs the Amazon Inspector agent**

    As described in the [Installing Amazon Inspector Agents](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_installing-uninstalling-agents.html) documentation, the following shell command installs the Amazon Inspector agent on an Amazon Linux-based EC2 instance. This command downloads a bash installation script from the AWS public repository and runs a bash install command to execute the script:
    ```
    wget https://inspector-agent.amazonaws.com/linux/latest/install
    bash install
    ```
    To find this command for other operating systems, see [Installing Amazon Inspector Agents](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_installing-uninstalling-agents.html).


2.  **Identify the command that starts the Amazon Inspector agent**

    Also described in the [Installing Amazon Inspector Agents](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_installing-uninstalling-agents.html) documentation, the following command starts the Amazon Inspector agent on an Amazon Linux-based EC2 instance:
    ```
    sudo /etc/init.d/awsagent start
    ```
    Again, to find this command for other operating systems, see [Amazon Inspector Agents](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_agents.html).

3.  **Create a script by concatenating the commands from the preceding two steps**

    By combining these two commands, you could construct a script to install and start the [Amazon Inspector Agent](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_agents.html). 

    And to make this script user-data compatible you would need to explicitly specify the shell at the beginning of the script. As a result, your script would look like this:

    ```
    #!/bin/bash
    wget https://inspector-agent.amazonaws.com/linux/latest/install
    bash install
    sudo /etc/init.d/awsagent start
    ```

4.  **Make the user-data script JSON compatible**

    Because we need to specify the user-data in JSON, and because specifying a newline character in JSON is complicated, we can simplify matters by replacing all the newline characters with `;`. You can also use third-party and online tools to escape all the `"` and newline characters. Once you are done, your user-data script should look like the one below **(For simplicity, just use this script in your JSON)**:

    ```
    "#!/bin/bash \n wget https://inspector-agent.amazonaws.com/linux/latest/install;bash install;/etc/init.d/awsagent start"
    ```
    
5.  **Create a JSON document of the metadata for all your Golden AMIs**

    We are now ready to construct our JSON metadata. Open your notepad or text editor and copy and paste in the JSON below, replacing: 
    * **instance-type-of-first-AMI** with the instance type you will use to launch the AMI to be inspected.
    * **AMI-ID-of-first-AMI** with the AMI id of your golden AMI.
    * **JSON-compatible-user-data-of-first-AMI** with the script in the previous step above.

    Use the following template to create your JSON document:
    
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
            "ami-id": "ami-0123456789abcd", 
            "userData": "#!/bin/bash \n wget https://inspector-agent.amazonaws.com/linux/latest/install;bash install;/etc/init.d/awsagent start"
         }
    ]
    ```

    **Note:**
    If you have multiple Golden AMIs you would like to inspect simply add another entry into the list as per below example.

    ```
    [	
        { 
            "instanceType": "t2.large", 
            "ami-id": "ami-0123456789abcd", 
            "userData": "#!/bin/bash \n wget https://inspector-agent.amazonaws.com/linux/latest/install;bash install;/etc/init.d/awsagent start"
        },	
        { 
            "instanceType": "t2.large", 
            "ami-id": "ami-abcd0123456789", 
            "userData": "#!/bin/bash \n wget https://inspector-agent.amazonaws.com/linux/latest/install;bash install;/etc/init.d/awsagent start"
         }
    ]
    ```

6.  Once you have everything setup, your entire JSON needs to be "stringified". This is because we will need to encapsulate this JSON once again inside a string value of our CloudFormation template.

    This process is a matter of escaping the `"` characters in your JSON with `\"`. You can use use online tools like https://onlinetexttools.com/json-stringify-text to do this or do this manually by hand.

    For simplicity you can just copy the string below and **replace** the `ami-00942d7cd4f3ca5c0` with **your own AMI ID**.

    ```
    "[{ \"instanceType\": \"t2.large\",\"ami-id\": \"ami-00942d7cd4f3ca5c0\", \"userData\": \"#!/bin/bash \\n wget https://inspector-agent.amazonaws.com/linux/latest/install;bash install;/etc/init.d/awsagent start\" }]"
    ```

    Now that you have created the stringified JSON for your golden AMI metadata, we will store this JSON document in a Systems Manager parameter. 
    
    But instead of specifying the parameter store through the AWS console, we will be defining the resource in [AWS CloudFormation](https://aws.amazon.com/cloudformation/)

### Create an AWS Systems Manager parameter using CloudFormation

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

In this step we will be **creating a AWS Systems Manager Parameter resource that will contain the stringified JSON we created in the previous steps**. To find out more about how to construct this resource, you can refer to documentation at https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ssm-parameter.html. 

Choose how you would like to construct the template from the options below:

<details><summary> **Option 1 - Build the resource manually with step-by-step instructions.**</summary>
<p>    

1. Open your text editor, create a file named `GoldenAMIParameters.yml`.
2. Create a `Resource:` template section. [Reference](https://docs.aws.amazon.com/en_pv/AWSCloudFormation/latest/UserGuide/template-anatomy.html) 
3. Create a resource named `AMIMetadata` of type `AWS::SSM::Parameter`.
4. The Name property of the parameter store is to be called `ContinuousAssessmentInput`.
5. The Type property of the parameter store is `String`.
6. The Value property of the parameter store is the stringified JSON data you created above.
7. The Description property is `Continuous golden AMI vulnerability assessment process metadata.`.

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
                Value: "[{ \"instanceType\": \"t2.large\",\"ami-id\": \"ami-00942d7cd4f3ca5c0\", \"userData\": \"#!/bin/bash \\n wget https://inspector-agent.amazonaws.com/linux/latest/install;bash install;/etc/init.d/awsagent start\" }]"
                Description: "Continuous golden AMI vulnerability assessment process metadata."
```

Copy and paste this into your `GoldenAMIParameters.yml` file, ensure to replace `ami-00942d7cd4f3ca5c0` with your own Golden AMI's Id.

</p>
</details>

### Deploy the CloudFormation template

Now that you've constructed the template, it's time to deploy the stack. Please follow the [Creating a Stack on the AWS CloudFormation Console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) guide to create your stack.

Specify `GoldenAMIParameters` as the stack name for simplicity.

Once you've launched your stack review the `Resources` Tab of the launch stack to identify the resources it's created. You should see an entry with `GoldenAMIParameter` as a Logical ID and `ContinuousAssessmentInput` as a Physical ID.

[Click here](https://ap-southeast-2.console.aws.amazon.com/systems-manager/parameters?region=ap-southeast-2) to go to the Systems Manager Parameter Store console, and confirm you see the `ContinuousAssesmentInput`.

**Congratulations!!!**

You have built a template from scratch! Not bad!
But we are just about to get started, let's move on to next stage to implement the actual solution stack.