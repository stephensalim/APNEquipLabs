+++
title = "Golden AMI Assessment Lab"
chapter = true
weight = 8
pre = "<b></b>"
+++

**Background**

As companies mature in their cloud journey, they implement layered security capabilities and practices in their cloud architectures. One such practice is to continually assess golden [Amazon Machine Images](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) (AMIs) for security vulnerabilities. AMIs provide the information required to launch an [Amazon EC2 instance](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Instances.html), which is a virtual server in the AWS Cloud. A _golden AMI_ is an AMI that contains the latest security patches, software, configuration, and software agents that you need to install for logging, security maintenance, and performance monitoring. You can build and deploy golden AMIs in your environment, but the AMIs quickly become dated as new vulnerabilities are discovered.

A security best practice is to perform routine vulnerability assessments of your golden AMIs to identify if any newly found vulnerabilities apply to them. If you identify a vulnerability, you can update your golden AMIs with the appropriate security patches, test the AMIs, and deploy the patched AMIs in your environment. In this Lab, you will learn how to use [Amazon Inspector](https://aws.amazon.com/inspector/) to set up a continuous vulnerability assessments to scan your golden AMIs routinely.

Amazon Inspector performs security assessments of [Amazon EC2](https://aws.amazon.com/ec2/) instances by using AWS managed [rules packages](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_rule-packages.html#InspectorRulePackages) such as the [Common Vulnerabilities and Exposures](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_cves.html) (CVEs) package. 

In this lab you will create a serverless Golden AMI vulnerability checker that will consolidate the assessment results and advise you of the next steps.

Furthermore, with this solution you can schedule the assessment to run regularly using [Amazon CloudWatch Events](https://aws.amazon.com/cloudwatch/).

**This Lab is based on the blog below:** 

<https://aws.amazon.com/blogs/security/how-to-set-up-continuous-golden-ami-vulnerability-assessments-with-amazon-inspector/>
