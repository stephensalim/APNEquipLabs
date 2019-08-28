---
title: "3. Launch EC2 Instance"
weight: 3
draft: false
---

1.  Go back to the EC2 Console.

2.  Click on **Launch Instance**
    
    ![](/Security Lab/images/image5.png)
    
3.  In the **Quick** Start section, select the "Amazon Linux AMI" and
    click **Select**
    
    ![](/Security Lab/images/image6.png)
    
    **PLEASE NOTE:** You must select "Amazon Linux AMI", **NOT** "Amazon
    Linux 2 AMI"! This lab will not work properly if you select Amazon
    Linux 2.

4.  Select the General purpose t2.micro instance type and click **Next:
    Configure Instance Details**
    
    ![](/Security Lab/images/image7.png)

5.  On the **Configure Instance Details** page, expand the **Advanced Details** section at the bottom of the page.
    
6. 	Type the following initialization script information into the UserData 	field. *(you can use Shift-Enter to create the necessary line break, or 	alternatively you could type this into Notepad and copy & paste the 	results)* (this will automatically install and start the stress tool & install Cloudwatch Agent).

	**Note:** Don't forget to replace <REPLACE WITH YOUR REGION> section in the script below with the region id where 	you run your EC2 instance.

   
	```
    #!/bin/sh

    # THIS COMMAND WILL INSTALL UPDATES ON OS
    yum -y update

    # THIS COMMAND WILL INSTALL AWSLOGS TOOL
    yum install -y awslogs
    sed -i -e 's/region = us-east-1/region = <REPLACE WITH YOUR REGION>/g' /etc/awslogs/awscli.conf

    # THIS COMMAND WILL CONFIGURE AWSLOGS TO LOG SECURE
    sed -i -e 's/messages/secure/g' /etc/awslogs/awslogs.conf

    sudo service awslogs start


	```

7. For the Network select either your "default VPC" or the vpc you created in 	the previous lab.

8. Select any subnet in the VPC (The subnet must be public facing, which means it has direct connection to the Internet Gateway).

9. 	Confirm "Auto-assign Public IP" is **Enabled** 

10. Ensure to select role name you created above in the **IAM role** section then 	click **Next: Add Storage**

	![](/Security Lab/images/image8.png)

11. Click **Next: Tag Instance** to accept the default Storage Device
    Configuration.
    
12. Next, choose a "friendly name" for your instance. This name, more
    correctly known as a tag, will appear in the console once the
    instance launches. It makes it easy to keep track of running
    machines in a complex environment. Named yours according to this
    format: "[Your Name] Server. Then click **Next: Configure Security
    Group**

13. Remove the Security Group rule with by clicking the "x" on the right
    so there are no rules. (You will not need to connect with this
    instance for this step). Then click **Review and Launch**
    
    ![](/Security Lab/images/image10.jpg)

14. Review your Instance Launch Configuration, and then click
    **Launch**.

	![](/Security Lab/images/image11.png)

15. In the drop down choose "Proceed without a keypair" and click
    **Launch Instances**.

	![](/Security Lab/images/image12.jpg)
	
16. Click the **View Instances** button in the lower right-hand portion
    of the screen to view the list of EC2 instances. Once your instance
    has launched, you will see your Server as well as the Availability
    Zone the instance is in.


