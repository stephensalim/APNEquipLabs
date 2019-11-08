---
title: "3. Change the Stack"
weight: 3
draft: false
---

Now that you have explored and deployed your VPC stack, 
let's modify the template and then update the stack to change the live environment.

1. Go back to the source CloudFormation template you created before.
2. Modify the sections of the source CloudFormation template as described, 
and save the changes as a new template file, for example Sample_VPC_CloudFormation_Updated.yml.


**Solution CloudFormation Template**

Click below to look at the solution CloudFormation template. If you like, you can simply copy and paste this into
a new file and update the stack. However, we recommend that in order to get the most out of this lab, you make the changes manually, following the instructions below.

<details><summary>CLICK HERE</summary>
<p>
```
AWSTemplateFormatVersion: '2010-09-09'
Parameters:
  vpccidr:
    Type: String
    MinLength: 9
    MaxLength: 18
    AllowedPattern: "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})"
    ConstraintDescription: Must be a valid CIDR range in the form x.x.x.x/16
    Default: 10.20.0.0/16
  psharedacidr:
    Type: String
    MinLength: 9
    MaxLength: 18
    AllowedPattern: "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})"
    ConstraintDescription: Must be a valid CIDR range in the form x.x.x.x/24
    Default: 10.20.0.0/24
  psharedbcidr:
    Type: String
    MinLength: 9
    MaxLength: 18
    AllowedPattern: "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})"
    ConstraintDescription: Must be a valid CIDR range in the form x.x.x.x/24
    Default: 10.20.1.0/24
  pvsharedacidr:
    Type: String
    MinLength: 9
    MaxLength: 18
    AllowedPattern: "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})"
    ConstraintDescription: Must be a valid CIDR range in the form x.x.x.x/24
    Default: 10.20.2.0/24
  pvsharedbcidr:
    Type: String
    MinLength: 9
    MaxLength: 18
    AllowedPattern: "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})"
    ConstraintDescription: Must be a valid CIDR range in the form x.x.x.x/24
    Default: 10.20.3.0/24
Resources:
  VPC:
    Type: "AWS::EC2::VPC"
    Properties:
      CidrBlock: !Ref vpccidr
  IGW:
    Type: "AWS::EC2::InternetGateway"
  S3AppBucket:
    DeletionPolicy: Retain
    Type: "AWS::S3::Bucket"
    Properties:
      AccessControl: PublicRead
      WebsiteConfiguration:
        ErrorDocument: index.html
        IndexDocument: index.html
  BucketPolicyApp:
    Type: "AWS::S3::BucketPolicy"
    Properties:
      Bucket: !Ref S3AppBucket
      PolicyDocument:
        Statement:
          -
            Sid: "ABC123"
            Action:
              - "s3:GetObject"
            Effect: Allow
            Resource: !Join ["", ["arn:aws:s3:::", !Ref S3AppBucket, "/*"]]
            Principal:
              AWS:
                - "*"
  GatewayAttach:
    Type: "AWS::EC2::VPCGatewayAttachment"
    Properties:
      InternetGatewayId: !Ref IGW
      VpcId: !Ref VPC
  SubnetPublicSharedA:
    Type: "AWS::EC2::Subnet"
    Properties:
      AvailabilityZone: !Select [0, !GetAZs ]
      CidrBlock: !Ref psharedacidr
      MapPublicIpOnLaunch: true
      VpcId: !Ref VPC
  SubnetPublicSharedB:
    Type: "AWS::EC2::Subnet"
    Properties:
      AvailabilityZone: !Select [1, !GetAZs ]
      CidrBlock: !Ref psharedbcidr
      MapPublicIpOnLaunch: true
      VpcId: !Ref VPC
  SubnetRouteTableAssociatePublicA:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Properties:
      RouteTableId: !Ref RouteTablePublic
      SubnetId: !Ref SubnetPublicSharedA
  SubnetRouteTableAssociatePublicB:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Properties:
      RouteTableId: !Ref RouteTablePublic
      SubnetId: !Ref SubnetPublicSharedB
  SubnetPrivateSharedA:
    Type: "AWS::EC2::Subnet"
    Properties:
      AvailabilityZone: !Select [0, !GetAZs ]
      CidrBlock: !Ref pvsharedacidr
      MapPublicIpOnLaunch: false
      VpcId: !Ref VPC
  SubnetPrivateSharedB:
    Type: "AWS::EC2::Subnet"
    Properties:
      AvailabilityZone: !Select [1, !GetAZs ]
      CidrBlock: !Ref pvsharedbcidr
      MapPublicIpOnLaunch: false
      VpcId: !Ref VPC
  SubnetRouteTableAssociatePrivateA:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Properties:
      RouteTableId: !Ref RouteTablePublic
      SubnetId: !Ref SubnetPrivateSharedA
  SubnetRouteTableAssociatePrivateB:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Properties:
      RouteTableId: !Ref RouteTablePublic
      SubnetId: !Ref SubnetPrivateSharedB
  RouteDefaultPublic:
    Type: "AWS::EC2::Route"
    DependsOn: GatewayAttach
    Properties:
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref IGW
      RouteTableId: !Ref RouteTablePublic
  RouteDefaultPrivateA:
    Type: "AWS::EC2::Route"
    Properties:
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NatGatewayA
      RouteTableId: !Ref RouteTablePrivateA
  RouteDefaultPrivateB:
    Type: "AWS::EC2::Route"
    Properties:
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NatGatewayB
      RouteTableId: !Ref RouteTablePrivateB
  RouteTablePublic:
    Type: "AWS::EC2::RouteTable"
    Properties:
      VpcId: !Ref VPC
  RouteTablePrivateA:
    Type: "AWS::EC2::RouteTable"
    Properties:
      VpcId: !Ref VPC
  RouteTablePrivateB:
    Type: "AWS::EC2::RouteTable"
    Properties:
      VpcId: !Ref VPC
  EIPNatGWA:
    DependsOn: GatewayAttach
    Type: "AWS::EC2::EIP"
    Properties:
      Domain: vpc
  EIPNatGWB:
    DependsOn: GatewayAttach
    Type: "AWS::EC2::EIP"
    Properties:
      Domain: vpc
  NatGatewayA:
    Type: "AWS::EC2::NatGateway"
    Properties:
      AllocationId: !GetAtt EIPNatGWA.AllocationId
      SubnetId: !Ref SubnetPublicSharedA
  NatGatewayB:
    Type: "AWS::EC2::NatGateway"
    Properties:
      AllocationId: !GetAtt EIPNatGWB.AllocationId
      SubnetId: !Ref SubnetPublicSharedB
Outputs:
  vpcid:
    Description: ID of Shared Infrastructure VPC
    Value: !Ref VPC
    Export: # added to export
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
    Export: # added to export
      Name: sharedinf-appbucketurl

```



</p>
</details>


1.  **Add Parameter Constraints:**

    -   Vpccidr
        -   Minimum length should be set to 9
        -   Maximum length should be set to 18
        -   Allowed pattern should be:
            `(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})`
        -   Add a constraint description
    -   Psharedacidr
        -   Minimum length should be set to 9
        -   Maximum length should be set to 18
        -   Allowed pattern should be:
            `(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})`
        -   Add a constraint description

    -   Psharedbcidr
        -   Minimum length should be set to 9
        -   Maximum length should be set to 18
        -   Allowed pattern should be:
            `(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})`
        -   Add a constraint description

    -   Pvsharedacidr
        -   Minimum length should be set to 9
        -   Maximum length should be set to 18
        -   Allowed pattern should be:
            `(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})`
        -   Add a constraint description

    -   Pvsharedbcidr
        -   Minimum length should be set to 9
        -   Maximum length should be set to 18
        -   Allowed pattern should be:
            `(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})`
        -   Add a constraint description

        Reference :<https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html>

        Before :

        ```
        Parameters:
        vpccidr:
            Type: String
            Default: 10.20.0.0/16
        psharedacidr:
            Type: String
            Default: 10.20.0.0/24
        psharedbcidr:
            Type: String
            Default: 10.20.1.0/24
        pvsharedacidr:
            Type: String
            Default: 10.20.2.0/24
        pvsharedbcidr:
            Type: String
            Default: 10.20.3.0/24
        ```
        After :

        ```
        Parameters:
        vpccidr:
            Type: String
            MinLength: 9
            MaxLength: 18
            AllowedPattern: "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})"
            ConstraintDescription: Must be a valid CIDR range in the form x.x.x.x/16
            Default: 10.20.0.0/16
        psharedacidr:
            Type: String
            MinLength: 9
            MaxLength: 18
            AllowedPattern: "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})"
            ConstraintDescription: Must be a valid CIDR range in the form x.x.x.x/24
            Default: 10.20.0.0/24
        psharedbcidr:
            Type: String
            MinLength: 9
            MaxLength: 18
            AllowedPattern: "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})"
            ConstraintDescription: Must be a valid CIDR range in the form x.x.x.x/24
            Default: 10.20.1.0/24
        pvsharedacidr:
            Type: String
            MinLength: 9
            MaxLength: 18
            AllowedPattern: "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})"
            ConstraintDescription: Must be a valid CIDR range in the form x.x.x.x/24
            Default: 10.20.2.0/24
        pvsharedbcidr:
            Type: String
            MinLength: 9
            MaxLength: 18
            AllowedPattern: "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})"
            ConstraintDescription: Must be a valid CIDR range in the form x.x.x.x/24
            Default: 10.20.3.0/24
        ```

2.  **Add a deletion policy:**

    -   Create a Deletion Policy for your S3 bucket, so that it will be retained on the stack being deleted.

        Reference : <https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-deletionpolicy.html>

        Before :

        ```
        S3AppBucket:
            Type: "AWS::S3::Bucket"
            Properties:
            AccessControl: PublicRead
            WebsiteConfiguration:
                ErrorDocument: index.html
                IndexDocument: index.html
        ```    

        After :

        ```
        S3AppBucket:
            DeletionPolicy: Retain
            Type: "AWS::S3::Bucket"
            Properties:
            AccessControl: PublicRead
            WebsiteConfiguration:
                ErrorDocument: index.html
                IndexDocument: index.html
        ```

3.  **Add an Outputs section, so that stack values are visible on the AWS CloudFormation Outputs tab:**

    -   vpcid
        -   Create a description of your output
        -   Reference your VPC, by using !Ref to obtain the value of the VPC ID

    -   natgatewayaid
        -   Create a description of your output
        -   Reference your NAT gateway A, using !Ref

    -   natgatewaybid
        -   Create a description of your output
        -   Reference your NAT gateway B, using !Ref

    -   appbucketurl
        -   Create a description of your output
        -   Use the !GetAtt intrinsic function with the WebsiteURL attribute to obtain the bucket URL from the bucket resource

    Reference : <https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/outputs-section-structure.html>

    Before :
    
    ```
    No Outputs are defined in the source template.
    ```

    After :

    ```
    Outputs:
        vpcid:
            Description: ID of Shared Infrastructure VPC
            Value: !Ref VPC
        natgatewayaid:
            Description: ID of NAT Gateway A
            Value: !Ref NatGatewayA
        natgatewaybid:
            Description: ID of NAT Gateway B
            Value: !Ref NatGatewayB
        appbucketurl:
            Description: Shared Infrastructure App Bucket
            Value: !GetAtt S3AppBucket.WebsiteURL
    ```

4.  **Add export values in Outputs section for Cross-Stack Reference:**

    -   vpcid
        -   Export your vpcid as 'sharedinf-vpc'
    -   appbucketurl
        -   Export your bucket URL as 'sharedinf-appbucketurl'

        Reference : <https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/outputs-section-structure.html>

        Before :

            ```
            Outputs:
            vpcid:
                Description: ID of Shared Infrastructure VPC
                Value: !Ref VPC
            natgatewayaid:
                Description: ID of NAT Gateway A
                Value: !Ref NatGatewayA
            natgatewaybid:
                Description: ID of NAT Gateway B
                Value: !Ref NatGatewayB
            appbucketurl:
                Description: Shared Infrastructure App Bucket
                Value: !GetAtt S3AppBucket.WebsiteURL
            ```

        After :

            ```
            Outputs:
            vpcid:
                Description: ID of Shared Infrastructure VPC
                Value: !Ref VPC
                Export: # added to export
                    Name: sharedinf-vpcid
            natgatewayaid:
                Description: ID of NAT Gateway A
                Value: !Ref NatGatewayA
            natgatewaybid:
                Description: ID of NAT Gateway B
                Value: !Ref NatGatewayB
            vpcid:
                Description: Shared Infrastructure App Bucket
                Value: !GetAtt S3AppBucket.WebsiteURL
                Export: # added to export
                    Name: sharedinf-appbucketurl
            ```

Other References:
===========

**Intrinsic functions:**

<https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html>


**Mappings:**

<https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/mappings-section-structure.html>

