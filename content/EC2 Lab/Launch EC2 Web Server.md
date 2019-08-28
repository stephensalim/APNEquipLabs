---
title: "2. Launch EC2 Web Server"
weight: 2
draft: false
---

In this example we will launch an Amazon Linux 2 instance, bootstrap
Apache/PHP, and install a basic web page that will display information
about our instance.

Sign into your AWS Management Console and choose EC2 from the Services
menu. [ ]{.underline}

1.  Click on Launch Instance

	![](/Detailed Instructions/EC2 Lab/images/image7.png)

8.  In the **Quick Start** section, select the first Amazon Linux 2 AMI
    and click **Select.**

    ![](/Detailed Instructions/EC2 Lab/images/image8.png)

9.  In the Choose Instance Type tab, select the t2.micro instance size
    and click **Next**.

10. On the **Configure Instance Details** page, expand the **Advanced
    Details** section, copy/paste the script below into the **User
    Data** field (this shell script will install Apache & PHP, start the
    web service, and deploy a simple web page). Click **Next.**

	```
	#!/bin/sh
	sudo su
	yum -y install httpd php mysql php-mysql
	chkconfig httpd on
	/etc/init.d/httpd -T
	cd /tmp
	wget
	http://us-east-1-aws-training.s3.amazonaws.com/self-paced-lab-4/examplefiles-as.zip
	unzip examplefiles-as.zip
	mv examplefiles-as/\* /var/www/html
	ls
	```

11. On this page you have the ability to modify or add storage and disk
    drives to the instance. For this lab, we will simply accept the
    storage defaults and click **Next.**

12. Here, we choose a "friendly name" for your instance by choosing
    'click to add a Name tag'. This name, more correctly known as a
    **tag**, will appear in the console once the instance launches. It
    makes it easy to keep track of running machines in a complex
    environment. Name yours as: "\[Your Name\] Web Server", and then
    click **Next**.

13. You will be prompted to create a new security group, which will be
    your firewall rules. On the assumption that we are building out a
    Web server, name your new security group "\[Your Name\] Web Tier",
    and confirm an existing SSH rule exists which allows TCP port 22
    from Anywhere. Click **Add Rule.**:

14. Select HTTP from the 'Type' dropdown menu, and confirm TCP port 80
    is allowed from Anywhere *(you'll notice, that "Anywhere is the same
    as '0.0.0.0/0')*. Click **Add Rule**.

    ![](/Detailed Instructions/EC2 Lab/images/image9.png)

15. Click the **Review and Launch** button after configuring the
    security group.

16. Review your cofiguration and choices, and then click **Launch**.

17. Select the key pair that you created in the beginning of this lab
    from the drop-down and check the \"I acknowledge\" checkbox. Then
    click the **Launch Instances** button.

18. Click the **View Instances** button in the lower righthand portion
    of the screen to view the list of EC2 instances. Once your instance
    has launched, you will see your Web Server as well as the
    Availability Zone the instance is in, and the publicly routable DNS
    name.

19. Click the checkbox next to your web server to view details about
    this EC2 instance.

	![](/Detailed Instructions/EC2 Lab/images/image10.png)
