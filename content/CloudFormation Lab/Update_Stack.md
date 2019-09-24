---
title: "4. Update Stack"
weight: 4
draft: false
---

Once you modified your existing template , you can use **Update Stack**
option to update your stack. To update select your Stack and click on
**Actions drop down and you will find Update Stack** option**.**

![](/CloudFormation Lab/images/media/image11.png)

In **Update**, Stack screen select browse your updated template. If you
have not figured out a solution yet Follow instruction in appendix
section to get template

*Lab\_Solution\_CloudFormation\_Module\_General\_ImmersionDay.json*.

![](/CloudFormation Lab/images/media/image12.png)

Now remaining steps are same as you followed in Create stack. Click Next
couple of time and you will land up to review summary screen, where you
need to click on **Update** button :

![](/CloudFormation Lab/images/media/image13.png)

Now you will find your stack status changed to **UPDATE\_IN\_PROGRESS**
and **Events** tab showing the activity performed using update stack.

![](/CloudFormation Lab/images/media/image14.png)

Once stack status changed to UPDATE\_COMPLETE status, you can browse to
**Outputs** tab and find out our changes has reflected now Outputs tab
has four values compare to earlier it was empty :

![](/CloudFormation Lab/images/media/image15.png)

Also, click on **CloudFormation** icon on the right top corner of the
screen and select **Exports** option**,** you will find two exported
value shown in here which can be utilized for cross-stack reference.

![](/CloudFormation Lab/images/media/image16.png)

To create a cross-stack reference, use the **Export** output field to
flag the value of a resource-output for export. Then, use the **Fn::
ImportValue** intrinsic function to import the value.

Appendix:
=========

**Initial AWS CloudFormation Template for lab exercise:**

Create a file
Lab\_Initial\_CloudFormation\_Module\_General\_ImmersionDay.yaml and
copy paste following code :

AWSTemplateFormatVersion: \'2010-09-09\'

Parameters:

vpccidr:

Type: String

Default: 10.20.0.0/16

psharedacidr:

Type: String

Default: 10.20.0.0/22

psharedbcidr:

Type: String

Default: 10.20.4.0/22

Resources:

VPC:

Type: \"AWS::EC2::VPC\"

Properties:

CidrBlock: !Ref vpccidr

IGW:

Type: \"AWS::EC2::InternetGateway\"

S3AppBucket:

Type: \"AWS::S3::Bucket\"

Properties:

AccessControl: PublicRead

WebsiteConfiguration:

ErrorDocument: index.html

IndexDocument: index.html

BucketPolicyApp:

Type: \"AWS::S3::BucketPolicy\"

Properties:

Bucket: !Ref S3AppBucket

PolicyDocument:

Statement:

\-

Sid: \"ABC123\"

Action:

\- \"s3:GetObject\"

Effect: Allow

Resource: !Join \[\"\", \[\"arn:aws:s3:::\", !Ref S3AppBucket,
\"/\*\"\]\]

Principal:

AWS:

\- \"\*\"

GatewayAttach:

Type: \"AWS::EC2::VPCGatewayAttachment\"

Properties:

InternetGatewayId: !Ref IGW

VpcId: !Ref VPC

SubnetPublicSharedA:

Type: \"AWS::EC2::Subnet\"

Properties:

AvailabilityZone: !Select \[0, !GetAZs \]

CidrBlock: !Ref psharedacidr

MapPublicIpOnLaunch: true

VpcId: !Ref VPC

SubnetPublicSharedB:

Type: \"AWS::EC2::Subnet\"

Properties:

AvailabilityZone: !Select \[1, !GetAZs \]

CidrBlock: !Ref psharedbcidr

MapPublicIpOnLaunch: true

VpcId: !Ref VPC

SubnetRouteTableAssociatePublicA:

Type: \"AWS::EC2::SubnetRouteTableAssociation\"

Properties:

RouteTableId: !Ref RouteTablePublic

SubnetId: !Ref SubnetPublicSharedA

SubnetRouteTableAssociatePublicB:

Type: \"AWS::EC2::SubnetRouteTableAssociation\"

Properties:

RouteTableId: !Ref RouteTablePublic

SubnetId: !Ref SubnetPublicSharedB

RouteDefaultPublic:

Type: \"AWS::EC2::Route\"

DependsOn: GatewayAttach

Properties:

DestinationCidrBlock: 0.0.0.0/0

GatewayId: !Ref IGW

RouteTableId: !Ref RouteTablePublic

RouteDefaultPrivateA:

Type: \"AWS::EC2::Route\"

Properties:

DestinationCidrBlock: 0.0.0.0/0

NatGatewayId: !Ref NatGatewayA

RouteTableId: !Ref RouteTablePrivateA

RouteDefaultPrivateB:

Type: \"AWS::EC2::Route\"

Properties:

DestinationCidrBlock: 0.0.0.0/0

NatGatewayId: !Ref NatGatewayB

RouteTableId: !Ref RouteTablePrivateB

RouteTablePublic:

Type: \"AWS::EC2::RouteTable\"

Properties:

VpcId: !Ref VPC

RouteTablePrivateA:

Type: \"AWS::EC2::RouteTable\"

Properties:

VpcId: !Ref VPC

RouteTablePrivateB:

Type: \"AWS::EC2::RouteTable\"

Properties:

VpcId: !Ref VPC

EIPNatGWA:

DependsOn: GatewayAttach

Type: \"AWS::EC2::EIP\"

Properties:

Domain: vpc

EIPNatGWB:

DependsOn: GatewayAttach

Type: \"AWS::EC2::EIP\"

Properties:

Domain: vpc

NatGatewayA:

Type: \"AWS::EC2::NatGateway\"

Properties:

AllocationId: !GetAtt EIPNatGWA.AllocationId

SubnetId: !Ref SubnetPublicSharedA

NatGatewayB:

Type: \"AWS::EC2::NatGateway\"

Properties:

AllocationId: !GetAtt EIPNatGWB.AllocationId

SubnetId: !Ref SubnetPublicSharedB

**Solution AWS CloudFormation Template to review at end of the lab:**

Create a file
Lab\_Solution\_CloudFormation\_Module\_General\_ImmersionDay.yaml and
copy paste following code :

AWSTemplateFormatVersion: \'2010-09-09\'

Parameters:

vpccidr:

Type: String

MinLength: 9

MaxLength: 18

AllowedPattern:
\"(\\\\d{1,3})\\\\.(\\\\d{1,3})\\\\.(\\\\d{1,3})\\\\.(\\\\d{1,3})/(\\\\d{1,2})\"

ConstraintDescription: Must be a valid CIDR range in the form x.x.x.x/16

Default: 10.20.0.0/16

psharedacidr:

Type: String

MinLength: 9

MaxLength: 18

AllowedPattern:
\"(\\\\d{1,3})\\\\.(\\\\d{1,3})\\\\.(\\\\d{1,3})\\\\.(\\\\d{1,3})/(\\\\d{1,2})\"

ConstraintDescription: Must be a valid CIDR range in the form x.x.x.x/22

Default: 10.20.0.0/22

psharedbcidr:

Type: String

MinLength: 9

MaxLength: 18

AllowedPattern:
\"(\\\\d{1,3})\\\\.(\\\\d{1,3})\\\\.(\\\\d{1,3})\\\\.(\\\\d{1,3})/(\\\\d{1,2})\"

ConstraintDescription: Must be a valid CIDR range in the form x.x.x.x/22

Default: 10.20.4.0/22

Resources:

VPC:

Type: \"AWS::EC2::VPC\"

Properties:

CidrBlock: !Ref vpccidr

IGW:

Type: \"AWS::EC2::InternetGateway\"

S3AppBucket:

DeletionPolicy: Retain

Type: \"AWS::S3::Bucket\"

Properties:

AccessControl: PublicRead

WebsiteConfiguration:

ErrorDocument: index.html

IndexDocument: index.html

BucketPolicyApp:

Type: \"AWS::S3::BucketPolicy\"

Properties:

Bucket: !Ref S3AppBucket

PolicyDocument:

Statement:

\-

Sid: \"ABC123\"

Action:

\- \"s3:GetObject\"

Effect: Allow

Resource: !Join \[\"\", \[\"arn:aws:s3:::\", !Ref S3AppBucket,
\"/\*\"\]\]

Principal:

AWS:

\- \"\*\"

GatewayAttach:

Type: \"AWS::EC2::VPCGatewayAttachment\"

Properties:

InternetGatewayId: !Ref IGW

VpcId: !Ref VPC

SubnetPublicSharedA:

Type: \"AWS::EC2::Subnet\"

Properties:

AvailabilityZone: !Select \[0, !GetAZs \]

CidrBlock: !Ref psharedacidr

MapPublicIpOnLaunch: true

VpcId: !Ref VPC

SubnetPublicSharedB:

Type: \"AWS::EC2::Subnet\"

Properties:

AvailabilityZone: !Select \[1, !GetAZs \]

CidrBlock: !Ref psharedbcidr

MapPublicIpOnLaunch: true

VpcId: !Ref VPC

SubnetRouteTableAssociatePublicA:

Type: \"AWS::EC2::SubnetRouteTableAssociation\"

Properties:

RouteTableId: !Ref RouteTablePublic

SubnetId: !Ref SubnetPublicSharedA

SubnetRouteTableAssociatePublicB:

Type: \"AWS::EC2::SubnetRouteTableAssociation\"

Properties:

RouteTableId: !Ref RouteTablePublic

SubnetId: !Ref SubnetPublicSharedB

RouteDefaultPublic:

Type: \"AWS::EC2::Route\"

DependsOn: GatewayAttach

Properties:

DestinationCidrBlock: 0.0.0.0/0

GatewayId: !Ref IGW

RouteTableId: !Ref RouteTablePublic

RouteDefaultPrivateA:

Type: \"AWS::EC2::Route\"

Properties:

DestinationCidrBlock: 0.0.0.0/0

NatGatewayId: !Ref NatGatewayA

RouteTableId: !Ref RouteTablePrivateA

RouteDefaultPrivateB:

Type: \"AWS::EC2::Route\"

Properties:

DestinationCidrBlock: 0.0.0.0/0

NatGatewayId: !Ref NatGatewayB

RouteTableId: !Ref RouteTablePrivateB

RouteTablePublic:

Type: \"AWS::EC2::RouteTable\"

Properties:

VpcId: !Ref VPC

RouteTablePrivateA:

Type: \"AWS::EC2::RouteTable\"

Properties:

VpcId: !Ref VPC

RouteTablePrivateB:

Type: \"AWS::EC2::RouteTable\"

Properties:

VpcId: !Ref VPC

EIPNatGWA:

DependsOn: GatewayAttach

Type: \"AWS::EC2::EIP\"

Properties:

Domain: vpc

EIPNatGWB:

DependsOn: GatewayAttach

Type: \"AWS::EC2::EIP\"

Properties:

Domain: vpc

NatGatewayA:

Type: \"AWS::EC2::NatGateway\"

Properties:

AllocationId: !GetAtt EIPNatGWA.AllocationId

SubnetId: !Ref SubnetPublicSharedA

NatGatewayB:

Type: \"AWS::EC2::NatGateway\"

Properties:

AllocationId: !GetAtt EIPNatGWB.AllocationId

SubnetId: !Ref SubnetPublicSharedB

Outputs:

vpcid:

Description: ID of Shared Infrastructure VPC

Value: !Ref VPC

Export: \# added to export

Name: sharedinf-vpcid

natgatewayaid:

Description: ID of NAT Gateway A

Value: !Ref NatGatewayA

natgatewaybid:

Description: ID of NAT Gateway B

Value: !Ref NatGatewayB

appbucketurl:

Description: Shared Infrastructure App Bucket

Value: !GetAtt S3AppBucket.WebsiteURL

Export: \# added to export

Name: sharedinf-appbucketurl
