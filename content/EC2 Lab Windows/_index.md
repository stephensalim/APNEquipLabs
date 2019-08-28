+++
title = "EC2 Lab (Windows)"
chapter = true
weight = 3
pre = "<b></b>"
+++

**Amazon Web Services** offers you the flexibility to run Microsoft Windows Server for as much or as little time as you need. You have many versions of Windows to choose from and the ability to specify server and network performance capabilities based on your requirements.

**Amazon Virtual Private Cloud (VPC**) lets you provision a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define. You have complete control over your virtual networking environment, including selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways.

**AWS Identity and Access Management (IAM)** enables you to manage access to AWS services and resources securely. Using IAM, you can create and manage AWS users and groups, and use permissions to allow and deny their access to AWS resources.

**Amazon Elastic Compute Cloud (Amazon EC2)** is a web service that provides resizable compute capacity in the cloud. Amazon EC2’s simple web service interface allows you to obtain and configure capacity with minimal friction. Amazon EC2 reduces the time required to obtain and boot new server instances to minutes, allowing you to quickly scale capacity, both up and down, as your computing requirements change. Amazon EC2 changes the economics of computing by allowing you to pay only for capacity that you actually use.

Microsoft and Amazon have jointly developed a set of Amazon Machine Images (AMIs) for some of the more popular Microsoft solutions. For more information on the available Windows AMIs go to: <https://aws.amazon.com/windows/resources/amis/>


In this lab, you will create a VPC and define a public network subnet within it. Then you will create an IAM policy to restrict EC2 instance launch to a specific region and assign this policy to a user. Finally you will launch a Windows EC2 instance and connect to it remotely via Remote Desktop Connect.

To complete the lab, you need the following **requirements**:

* Microsoft Remote Desktop Connect installed on your local computer.
* An AWS Account
* An AWS IAM user with rights to create a Virtual Private Cloud (VPC) in your AWS account and ability to create Routes/Internet Gateway.
* An AWS IAM user with privileges to create/modify EC2 instances and create IAM users.