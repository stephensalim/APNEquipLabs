---
title: "1. Solutions Overview"
weight: 1
draft: false
---

![](/AMI Inspector Lab/images/Main.png)

Here is the overview on how this solution works :

1.  A scheduled CloudWatch Events event will trigger the [AWS Lambda](http://aws.amazon.com/lambda/) function called `StartContinuousAssessment`.

2.  This function will then pulls metadata information containing the golden AMI ID, Instance type, and instructions to prepare the EC2 instance for inspection from [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)

3.  For each of the AMI specified in the metadata, `StartContinuousAssessment` Lambda function will create an EC2 instance.

4.  Levaraging [EC2 User Data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) It will then runs the instructions to install [Amazon Inspector](https://aws.amazon.com/inspector/) agent allowing the instance to to be inspected.

5.  The function will then execute the assessment activity from [Amazon Inspector](https://aws.amazon.com/inspector/) service towards the instance.
        
6.  After [Amazon Inspector](https://aws.amazon.com/inspector/) completes the Inspection, it will then send a notification the the [SNS Topic](https://docs.aws.amazon.com/sns/latest/dg/welcome.html) triggering the `AnalyzeInspectorFindings` [AWS Lambda](http://aws.amazon.com/lambda/) function.

7. `AnalyzeInspectorFindings` will then tag the inspection result with relevant information about the AMI, Clean up the EC2 instance used to inspect and finally send an email the user with the total of vunerability findings in the AMI.

Now that we know at a high level how this solution works, lets start building them, on to the next page :)