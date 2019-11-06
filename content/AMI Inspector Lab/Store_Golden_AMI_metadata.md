---
title: "3. Store Golden AMI metadata."
weight: 3
draft: false
---

 ![](/AMI Inspector Lab/images/StoreMetaParam.png)

In this section we will read the golden AMI metadata from a parameter stored in the Systems Manager Parameter Store. The metadata will be in JSON format and must contain the following information for each golden AMI:

* The **Ami-Id** is the golden AMI id that we tagged in the previous step ( Hopefully you've taken note of that, otherwise please review the previous step ).
* The **InstanceType** is the EC2 instance type our solution will use to launch the golden AMI with and run the inspection on.
* The **UserData** section will contain our bootstrap instructions to install AWS Inspector Agent.

    **Note:** 

    You must use a compatible InstanceType that is supported by your AMI, to find the compatible `InstanceType` for your golden AMI follow below steps. 
    (But for simplicity you can use t2.large). 
    
    Each AMI has a list of compatible `InstanceTypes`. The `InstanceType` is a required parameter for launching an EC2 instance from a golden AMI. To find a compatible `InstanceType` for your golden AMI:

    1.  Sign in to the [AWS Management Console](https://console.aws.amazon.com/console/home) and navigate to the [EC2 console](https://console.aws.amazon.com/ec2/).
    2.  Choose **Launch Instance**. On the **Choose an Amazon Machine Image** (**AMI**) page, choose **My AMIs**.
    3.  Type the **AMI ID** that you noted in Step A in the **Search my AMIs** box, and then choose **Enter**.
    4.  The search result will contain your golden AMI. To choose it, choose **Select**.
    5.  Locate any available **Instance Type** and then note the corresponding value in the **Type** column.
    6.  Choose **Cancel**.
    Amazon Inspector will launch the chosen `InstanceType` every time the vulnerability assessment runs.

For the UserData section you will need to create a `user-data` script to install and start the Amazon Inspector agent.
The `user-data` script automates the installation of software packages when an EC2 instance launches for the first time. 
In this step, you will create an operating system specific, JSON-compatible user-data script that installs and starts the Amazon Inspector agent.

1.  **Identify the command that installs the Amazon Inspector agent**
    Based on [Installing Amazon Inspector Agents](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_installing-uninstalling-agents.html), the following shell command installs the Amazon Inspector agent on an Amazon Linux-based EC2 instance.
    ```
    wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install
    bash install
    ```
    To find this command for other operating systems, see [Installing Amazon Inspector Agents](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_installing-uninstalling-agents.html).


2.  **Identify the command that starts the Amazon Inspector agent**
    The following shell command starts the Amazon Inspector agent on an Amazon Linux-based EC2 instance.
    ```
    sudo /etc/init.d/awsagent start
    ```
    To find this command for other operating systems, see [Amazon Inspector Agents](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_agents.html).

3.  **Create a script by concatenating the commands from the preceding two steps**

    The following is a sample concatenated script for the Amazon Linux operating system that installs and starts an Amazon Inspector agent.
    ```
    wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install
    bash install
    sudo /etc/init.d/awsagent start
    ```


4.  **Make the script user-data compatible**

    Based on [Running Commands on Your Linux Instance at Launch](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html#user-data-api-cli), you make a Linux shell script user-data compatible by prefixing it with a `#!/bin/bash`. In this step, you add the `#!/bin/bash` prefix to the script from the preceding step. The following is the user-data compatible version of the script from the preceding step.

    ```
    #!/bin/bash
    wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install
    bash install
    sudo /etc/init.d/awsagent start
    ```
    To make your script user-data compatible for Windows, see [Running Commands on Your Windows Instance at Launch](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ec2-windows-user-data.html).

    The `user-data` script provided in the JSON metadata must be JSON-compatible, which you will do next.

5.  **Make the user-data script JSON compatible**

    To make the user-data script JSON compatible, replace all the commands in step 4 into a one line of string.
    You can use third party online tools to escape all the " signs and new line characters.
    But to make things easy for you I have created a converted version for you below. 

    **JSON-compatible-user-data-of-first-AMI**
    ```
    "#!/bin/bash \n wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install;bash install;/etc/init.d/awsagent start"
    ```
    
6.  **Create a JSON document of metadata of all your golden AMIs.**

    Open your notepad or text editor copy and paste the JSON below and replace 
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

7.  Once you have everything setup your JSON needs to be converted into a string.
    This is just a matter of converting your JSON and escaping the " signs.
    Again for simplicity you can just copy the string below and replace the ami-0fe2fc7acf5400b25 with your prefered ami.

    ```
    "[{ \"instanceType\": \"t2.large\",\"ami-id\": \"ami-0fe2fc7acf5400b25\", \"userData\": \"#!/bin/bash \\n wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install;bash install;/etc/init.d/awsagent start\" }]"
    ```

    Now that you have created the JSON document of your golden AMIs, you will store the JSON document in a Systems Manager parameter. The `StartContinuousAssessment` Lambda function will read the metadata from this parameter.

8.  **Create an AWS Systems Manager parameter using CloudFormation**
    
    Open your notepad / text editor, create a file named `GoldenAMIParameters.yml`

    Reference : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ssm-parameter.html

    * Create a resource named `AMIMetadata` of type `AWS::SSM::Parameter`.
    * The name properties of the parameter store should be called `ContinuousAssessmentInput`
    * The type properties of the parameter store is `String`
    * The value properties of the parameter store is the String you've created in step 7 above.
    * The Description properties is `Continuous golden AMI vulnerability assessment process metadata.`

    <details><summary>CLICK HERE to see the solution</summary>
    <p>

    ```
    Resources:
    BasicParameter:
        Type: "AWS::SSM::Parameter"
        Properties:
        Name: "ContinuousAssessmentInput_new"
        Type: "String"
        Value: "[{ \"instanceType\": \"t2.large\",\"ami-id\": \"ami-0e2b940b603bf07f3\", \"userData\": \"#!/bin/bash \\n wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install;bash install;/etc/init.d/awsagent start\" }]"
        Description: "Continuous golden AMI vulnerability assessment process metadata."
    ```

    </p>
    </detail>