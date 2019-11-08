+++
title = "CloudFormation Lab"
chapter = true
weight = 8
pre = "<b></b>"
+++

This lab will walk the user through using AWS CloudFormation to
create a VPC with public and private subnets. Then, each of the objects 
created by the AWS CloudFormation template will be described, and you will
be guided through the process of launching an AWS CloudFormation Stack from
the template, to create a VPC with public and private VPC subnets, 
a RouteTable, an Elastic IP NAT Gateway, and an S3 bucket.

The lab is broken down into the following high-level steps:

- Explore the initial AWS CloudFormation template   
- Explore the different VPC objects and what they mean
- Launch an AWS CloudFormation template by creating a Stack from Console
- Export the VPC ID, the NAT Gateway ID and the S3 bucket URL to AWS CloudFormation Stack outputs tab

The lab will provide an initial template for users to explore. After
creating the VPC stack from an initial template, users will need to 
complete the required steps to reach the final objective.

**Note:** Screenshots are provided to guide you through the steps in the
lab. The elements that you will create (e.g. VPC, NAT Gateway, EIP) will
be unique to your account, so please bear in mind that resources such as 
the VPC ID that you see in the console will not necessarily match the 
screenshots. Also, please be aware that the console may appear slightly 
different from the screenshots, but the the steps remain the same. 
