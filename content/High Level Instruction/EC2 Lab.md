---
title: "EC2 Lab"
weight: 2
draft: false
---

**Amazon Elastic Compute Cloud (Amazon EC2**) is a web service that provides
resizable compute capacity in the cloud. Amazon EC2's simple web service
interface allows you to obtain and configure capacity with minimal
friction. Amazon EC2 reduces the time required to obtain and boot new
server instances to minutes, allowing you to quickly scale capacity,
both up and down, as your computing requirements change. Amazon EC2
changes the economics of computing by allowing you to pay only for
capacity that you actually use.

This lab will walk you through launching, configuring, and customizing
an EC2 web server using the AWS Management Console.

**Challange**

1. 	Create a new Key Pair
2. 	Download and change the private key to read only mode (chmod 400). (Linux /Mac)
	Convert the private key to support putty (Windows)
3. 	Create a new EC2 instance, in a public facing subnet.
4. 	Use Amazon Linux AMI 2018.03.0 Amazon Machine Image.
5. 	Use t2.micro EC2 instance size.
6.	Bootstrap the EC2 instance using the following bash script

		```
		#!/bin/sh
		sudo su
		yum -y install httpd php mysql php-mysql
		chkconfig httpd on
		/etc/init.d/httpd –T
		cd /tmp
		wget http://us-east-1-aws-training.s3.amazonaws.com/self-paced-lab-4/examplefiles-as.zip
		unzip examplefiles-as.zip
		mv examplefiles-as/* /var/www/html
		ls 
		```
		
7.	Apply appropriate Security Group rule and Network Access List to allow public HTTP access.

**Additional challange / question**

1. Create an Amazon Machine image from the EC2 web server.
2. Configure snapshot lifecycle manager.
3. How could you make the WebServer elastic?
4. How could you make the WebServer balancing across several instance?

**Documentation References:**

* [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)
* [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/launching-instance.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/launching-instance.html)
* [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)
* [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html)
* [https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)
* [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html)
* [https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-getting-started.html](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-getting-started.html)


