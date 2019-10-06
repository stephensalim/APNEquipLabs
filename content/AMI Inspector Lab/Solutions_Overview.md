---
title: "1. Solutions Overview"
weight: 1
draft: false
---

![Solution diagram showing how this post's solution works](https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2017/12/15/KW_1217_diagram.png "Solution diagram showing how this post's solution works")

Here’s how this solution works :

1.  A scheduled CloudWatch Events event triggers the `StartContinuousAssessment` [AWS Lambda](http://aws.amazon.com/lambda/) function, which starts the security assessment of your golden AMIs.
2.  The `StartContinuousAssessment` Lambda function performs the following actions:

    It reads a JSON parameter stored in the [AWS Systems Manager](http://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html) (Systems Manager) Parameter Store. This JSON parameter contains the following metadata for each golden AMI:
        
      1.  `InstanceType` – A valid instance-type for launching an EC2 instance of the golden AMI.
      2.  `Ami-Id` – The ID of the golden AMI.
      3.  `UserData` – An operating system–compatible [user-data script](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) for installing the [Amazon Inspector agent](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_agents.html).

3.  For each AMI specified in the JSON parameter, the Lambda function creates an EC2 instance. When each instance starts, it installs the Amazon Inspector agent by using the user-data script provided in the JSON. The Lambda function then copies each golden AMI’s [tags](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html) (you will assign custom metadata in the form of tags to each golden AMI when you set up the solution) to the corresponding EC2 instance. The function also adds a tag with the `key` of `continuous-assessment-instance` and `value` as `true`. This tag identifies EC2 instances that require regular security assessments. The Lambda function copies the AMI’s tags to the instance (and later, to the security findings found for the instance) to help you identify the golden AMIs for each security finding. After you analyze security findings, you can patch your golden AMIs.

4.  The first time the `StartContinuousAssessment` function runs, it creates:
    * An [Amazon Inspector assessment target](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_applications.html): The target identifies EC2 instances to assess by using the `continuous-assessment-instance` tag.
    * An [Amazon Inspector assessment template](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_assessments.html#inspector-assessment-templates): The template contains a reference to the Amazon Inspector assessment target created in the preceding step and the following AWS managed rules packages to evaluate:
      *   [Common Vulnerabilities and Exposures](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_cves.html) (CVEs)
      *   [Center for Internet Security (CIS) Benchmarks](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_cis.html)
      *   [AWS Security Best Practices](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_security-best-practices.html)
      *   [Runtime Behavior Analysis](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_runtime-behavior-analysis.html)

    For subsequent assessments, the `StartContinuousAssessment` function reuses the target and the template created during the first run of `StartContinuousAssessment` function.

    **Note:** Amazon Inspector can start an assessment only after it finds at least one running Amazon Inspector agent. To allow EC2 instances to boot and the Amazon inspector agent to start, the Lambda function waits four minutes. Because the assessment runs for approximately one hour and boot time for EC2 instances typically takes a few minutes, all Amazon Inspector agents start before the assessment ends.

5.  The Lambda function then runs the assessment. The Amazon Inspector agents collect behavior and configuration data, and pass it to Amazon Inspector. Amazon Inspector analyzes the data and generates [Amazon Inspector findings](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_findings.html), which are possible security findings you may need to address.
        
6.  After the Lambda function completes the assessment, Amazon Inspector publishes an assessment-completion notification message to an [Amazon SNS](https://aws.amazon.com/sns/) topic called `ContinuousAssessmentCompleteTopic`. SNS uses _topics_, which are communication channels for sending messages and subscribing to notifications.

7. The notification message published to SNS triggers the `AnalyzeInspectorFindings` Lambda function, which performs the following actions:
  * Associates the tags of each EC2 instance with security findings found for that EC2 instance. This enables you to identify the security findings using the `app-name` tag you specified for your golden AMIs. You can use the information provided in the findings to patch your golden AMIs.
  * Terminates all instances associated with the `continuous-assessment-instance=true` tag.
  * Aggregates the number of findings found for each EC2 instance by severity and then publishes a consolidated result to an SNS topic called `ContinuousAssessmentResultsTopic`.
