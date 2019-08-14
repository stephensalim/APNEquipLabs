---
title: "EC2 Lab"
weight: 2
draft: true
---


**Challange**

1. Create a new Key Pair
2. Download and change mode to read only (chmod 400)
3. Create a new EC2 instance
4. Amazon Linux 2
	* T2.micro
	* Bootstrap instance using 

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
		
	* Apply appropriate 
	* Security groups
	* NACL 

**Additional challange**

1. Create an AMI out of your running WebServer
2. Configure snapshot lifecycle manager
3. How could you make the WebServer  elastic?
4. How could you make the WebServer balancing across several instance?

**Documentation References:**

* [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)
* [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/launching-instance.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/launching-instance.html)
* [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)
* [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html)
* [https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)
* [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html)
* [https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-getting-started.html](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-getting-started.html)


