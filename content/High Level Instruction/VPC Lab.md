---
title: "VPC Lab"
weight: 1
draft: true
---




**Challange**

1. Use the VPC Wizard to create a new VPC
	* Use both Public and Private subnets
2. Create an Elastic IP
3. Review your new VPC and draw out the complete deployment including:
	* Subnets
	* IGW
	* NAT Gateway
	* Security groups
	* NACL

**Additional challange**

1. Create a Public EC2 instance into the new VPC
	* Use t2.micro and amazon Linux 2 AMI assign network to new VPC
2. Test public access
	* Assign a new Elastic IP to the instance
	* Ensure security group and NACL allow EC2 to communicate
3. Is there another way to achieve public access without an elastic IP?
4. Clean up your account
	* Terminate EC2 instance
	* Release any Elastic IP’s
	* Delete NAT Gateway and the VPC

**Documentation References:**

* [https://docs.aws.amazon.com/directoryservice/latest/admin-guide/gsg_create_vpc.html#create_vpc](https://docs.aws.amazon.com/directoryservice/latest/admin-guide/gsg_create_vpc.html#create_vpc)
* [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)
* [https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html)
* [https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)
* [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/launching-instance.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/launching-instance.html)
* [https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)
* [https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
* [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html)
