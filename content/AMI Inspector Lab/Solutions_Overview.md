---
title: "1. Solutions Overview"
weight: 1
draft: false
---

![](/AMI Inspector Lab/images/Main.png)

---

**IMPORTANT NOTE:** 

For this solution to work, you must deploy all resouces AWS Region where you build your golden AMIs. 
This region must have [Amazon Inspector](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_supported_os_regions.html#inspector_supported-regions) available. 
For best lab experience, please choose Sydney Region **( ap-southeast-2)**.

---

Here an overview on how our solutions works:

1.  A scheduled [CloudWatch Events](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html) will trigger the [AWS Lambda](http://aws.amazon.com/lambda/) function called `StartContinuousAssessment` every morning at 6 AM. This will essentially acts like your cron task (Linux) or scheduled task (Windows) that will executes on a regular basis.

2.  `StartContinuousAssessment` [AWS Lambda](http://aws.amazon.com/lambda/) function will first gather information about which Golden AMI it should asses. This is done by downloading metadata information about the GoldenAMI that we store inside [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html). AWS Systems Manager Parameter Store provides secure, hierarchical storage for configuration data management and secrets management. 

3.  For each of the GoldenAMI specified in the metadata, `StartContinuousAssessment` function will then launch an EC2 Instance from the AMI.

4.  This EC2 instance will then levarage [EC2 User Data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) to bootstrap installation & starting of [Amazon Inspector](https://aws.amazon.com/inspector/) agent. This agent is needed for Amazon Inspector Service to inspect the EC2 at the Operating system layer.

5.  `StartContinuousAssessment` function will then execute the activity from [Amazon Inspector](https://aws.amazon.com/inspector/) to kick off the inspection of the EC2 instance security posture according to [CIS](https://docs.aws.amazon.com/inspector/latest/userguide/inspector_cis.html) and [CVE](https://docs.aws.amazon.com/inspector/latest/userguide/inspector_cves.html)
        
6.  Once [Amazon Inspector](https://aws.amazon.com/inspector/) completes the Inspection, a notification will then be sent to the [SNS Topic](https://docs.aws.amazon.com/sns/latest/dg/welcome.html) forwarding message to the email subscriber of the topic, as well as triggering the `AnalyzeInspectorFindings` [AWS Lambda](http://aws.amazon.com/lambda/) function where the next activity will takes place.

7. `AnalyzeInspectorFindings` function will tag [Amazon Inspector](https://aws.amazon.com/inspector/) result with relevant information about the AMI & EC2 Instance for context. It will then also clean up the temporary EC2 instance that was launched and used by Amazon Inspector as a medium to inspect the GoldenAMI posture.

Now that weare clear about this, lets start building them ! On to the next page !

**[`^0^]/** 