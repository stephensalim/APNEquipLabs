---
title: "3. Store Golden AMI metadata."
weight: 3
draft: false
---

This solution reads golden AMI metadata from a parameter stored in the Systems Manager Parameter Store. The metadata must be in JSON format and must contain the following information for each golden AMI:

*   `Ami-Id`
*   `InstanceType`
*   `UserData`

**Step A:** Find the AMI ID of your golden AMI.

An AMI ID uniquely identifies an AMI in an AWS Region and is a required parameter for launching an EC2 instance from a golden AMI. To find the AMI ID of your golden AMI:

1.  Sign in to the [AWS Management Console](https://console.aws.amazon.com/console/home) and navigate to the [EC2 console](https://console.aws.amazon.com/ec2/).
2.  In the navigation pane, choose **AMIs**.
3.  Choose your AMI from the list and then note the corresponding value in the **AMI ID** column.

**Step B:** Find a compatible `InstanceType` for your golden AMI.

Each AMI has a list of compatible `InstanceTypes`. The `InstanceType` is a required parameter for launching an EC2 instance from a golden AMI. To find a compatible `InstanceType` for your golden AMI:

1.  Sign in to the [AWS Management Console](https://console.aws.amazon.com/console/home) and navigate to the [EC2 console](https://console.aws.amazon.com/ec2/).
2.  Choose **Launch Instance**. On the **Choose an Amazon Machine Image** (**AMI**) page, choose **My AMIs**.
3.  Type the **AMI ID** that you noted in Step A in the **Search my AMIs** box, and then choose **Enter**.
4.  The search result will contain your golden AMI. To choose it, choose **Select**.
5.  Locate any available **Instance Type** and then note the corresponding value in the **Type** column.
6.  Choose **Cancel**.

**Note:** Amazon Inspector will launch the chosen `InstanceType` every time the vulnerability assessment runs.

**Step C:** Create the `user-data` script to install and start the Amazon Inspector agent.

The `user-data` script automates the installation of software packages when an EC2 instance launches for the first time. In this step, you create an operating system specific, JSON-compatible user-data script that installs and starts the Amazon Inspector agent.

1.  Identify the command that installs the Amazon Inspector agent

Based on [Installing Amazon Inspector Agents](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_installing-uninstalling-agents.html), the following shell command installs the Amazon Inspector agent on an Amazon Linux-based EC2 instance.

<div class="hide-language">

    wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install
    bash install

</div>

To find this command for other operating systems, see [Installing Amazon Inspector Agents](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_installing-uninstalling-agents.html).

1.  Identify the command that starts the Amazon Inspector agent

The following shell command starts the Amazon Inspector agent on an Amazon Linux-based EC2 instance.

    sudo /etc/init.d/awsagent start

To find this command for other operating systems, see [Amazon Inspector Agents](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_agents.html).

1.  Create a script by concatenating the commands from the preceding two steps

The following is a sample concatenated script for the Amazon Linux operating system that installs and starts an Amazon Inspector agent.

    wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install
    bash install
    sudo /etc/init.d/awsagent start

1.  Make the script user-data compatible

Based on [Running Commands on Your Linux Instance at Launch](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html#user-data-api-cli), you make a Linux shell script user-data compatible by prefixing it with a `#!/bin/bash`. In this step, you add the `#!/bin/bash` prefix to the script from the preceding step. The following is the user-data compatible version of the script from the preceding step.

    #!/bin/bash
    wget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install
    bash install
    sudo /etc/init.d/awsagent start

To make your script user-data compatible for Windows, see [Running Commands on Your Windows Instance at Launch](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ec2-windows-user-data.html).

The `user-data` script provided in the JSON metadata must be JSON-compatible, which you will do next.

1.  Make the user-data script JSON compatible

To make the user-data script JSON compatible, you must replace all new-line characters with a `\r\n\r\n` sequence. The following is the JSON-compatible user-data script that you specify for your Amazon Linux-based golden AMI in Step D.

<span style="text-decoration: underline">**`JSON-compatible-user-data-for-Amazon-Linux-AMI`**</span>

    #!/bin/bash \r\n\r\nwget https://d1wk0tztpsntt1.cloudfront.net/linux/latest/install \r\n\r\nbash install \r\n\r\nsudo /etc/init.d/awsagent start

Repeat Steps A, B, and C to find the `Ami Id`, `InstanceType`, and `UserData` for each of your golden AMIs. When you have this metadata, you can create the JSON document of metadata for all your golden AMIs. The `StartContinuousAssessment` Lambda function reads this JSON to start golden AMI vulnerability assessments.

**Step D**: Create a JSON document of metadata of all your golden AMIs.

Use the following template to create a JSON document:

<div class="hide-language">

    [	
        { 
            "instanceType": "instance-type-of-first-AMI", 
            "ami-id": "AMI-ID-of-first-AMI", 
            "userData": "JSON-compatible-user-data-of-first-AMI"
         },
        { 
            "instanceType": "instance-type-of-second-AMI",
            "ami-id": "AMI-ID-of-second-AMI",
            "userData": "JSON-compatible-user-data-of-second-AMI" 
        }
    ]

</div>

Replace all <span style="color: #ff0000">**placeholder values**</span> with values corresponding to your first golden AMI. If your golden AMI is Amazon Linux-based, you can specify the `userData` as the `JSON-compatible-user-data-for-Amazon-Linux-AMI` from Step C.5\. Next, replace the <span style="color: #0000ff">**placeholder values**</span> for your second golden AMI. You can add more entries to your JSON document, if you have more than two golden AMIs.

**Note****:** The total number of characters in the JSON document must be fewer than or equal to 4,096 characters, and the number of golden AMIs must be fewer than 500\. You must verify whether your account has permissions to run one on-demand EC2 instance for each of your golden AMIs. For information about how to verify service limits, see [Amazon EC2 Service Limits](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-resource-limits.html).

Now that you have created the JSON document of your golden AMIs, you will store the JSON document in a Systems Manager parameter. The `StartContinuousAssessment` Lambda function will read the metadata from this parameter.

**Step E:** Store the JSON in a Systems Manager parameter.

To store the JSON in a Systems Manager parameter:

1.  Sign in to the [AWS Management Console](https://console.aws.amazon.com/console/home) and navigate to the [EC2 console](https://console.aws.amazon.com/ec2/).
2.  Expand **Systems Manager Shared Resources** in the navigation pane, and then choose **Parameter Store**.
3.  Choose **Create Parameter**.
4.  For **Name**, type `ContinuousAssessmentInput`.
5.  In the **Description** field, type `Continuous golden AMI vulnerability assessment process metadata`.
6.  For **Type**, choose **String**.
7.  Paste the JSON that you created in **Step D** in the **Value** field.
8.  Choose **Create Parameter**. After the system creates the parameter, choose **Close**.

To set up the remaining components required to run assessments, you will run a CloudFormation template and perform the configuration explained in the next section.

