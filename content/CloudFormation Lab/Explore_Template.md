---
title: "1. Exploring the template"
weight: 1
draft: false
---

Please download the **Source CloudFormation Template** file. You can use
any text editor to explore the different elements of the VPC defined by
the template. To make it easy for you, can copy and paste the entire template 
into any YAML friendly text editor and collapse/expand the sections as required.

Some suggested editors:

* Notepad++ (https://notepad-plus-plus.org/downloads/)
* VisualStudioCode. (https://code.visualstudio.com/)


**Source CloudFormation Template**

Copy this template for the next section.

<details><summary>CLICK HERE</summary>
<p>


```
AWSTemplateFormatVersion: '2010-09-09'
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

Resources:
  VPC:
    Type: "AWS::EC2::VPC"
    Properties:
      CidrBlock: !Ref vpccidr
  IGW:
    Type: "AWS::EC2::InternetGateway"
  S3AppBucket:
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

```


</p>
</details>



You will notice following resources in the initial AWS CloudFormation
Template:

-   VPC

    ```
    ...
    VPC:
      Type: "AWS::EC2::VPC"
      Properties:
        CidrBlock: !Ref vpccidr
    ...
    ```

    **Note:**

    Notice that the value of CidrBlock is a reference to `vpccidr`. This is essentially the Parameters that you specified under the `Parameters` section of the template. Parameters are specified under it's own section outside (same level of) Resources. Open up the source CloudFormation template, and take note of `vpccidr` under Parameters section. This reference are linked using [!Ref intrinsic function](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html)


-   Internet Gateway

    ```
    ...
    IGW:
      Type: "AWS::EC2::InternetGateway"
      
    ...

    GatewayAttach:
      Type: "AWS::EC2::VPCGatewayAttachment"
      Properties:
        InternetGatewayId: !Ref IGW
        VpcId: !Ref VPC
    ...
    ```


    **Note:**

    Other linking between Parameters and Resource values, [!Ref intrinsic function](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html) could also be used to Link the value of other resource you specified. In this section you will see, InternetGatewayId has a value of !Ref to IGW which is the InternetGateway resource. Every resource in cloudformation will have values you can reference, weather it's the name / arn (it varies from resource to resource). To find out what value the resource provides for !Ref refer to the `Return Values` `Ref` section of each resource documentation, e.g : https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-internetgateway.html


-   S3 bucket

    ```
    ...
      S3AppBucket:
        Type: "AWS::S3::Bucket"
        Properties:
          AccessControl: PublicRead
          WebsiteConfiguration:
              ErrorDocument: index.html
              IndexDocument: index.html
    ...
    ```

-   Two public subnets with corresponding route tables
    ```
    ...
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
    ...

    ...
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
    RouteDefaultPublic:
        Type: "AWS::EC2::Route"
        DependsOn: GatewayAttach
        Properties:
          DestinationCidrBlock: 0.0.0.0/0
          GatewayId: !Ref IGW
          RouteTableId: !Ref RouteTablePublic
    ...

    ```

    **Note:**

    In this section we see the use of Intrinsic Function !Select !GetAZs in action `: !Select [0, !GetAZs ]`. 
    !Select intrinsic functions, gives your the capability to select a value from a list that is placed as the input. 
    In this example the input towards !Select is another intrinsic function called !GetAZs that returns a list of Avalability Zones names within the region of the stack e.g: ap-southeast-1a, ap-southeast-2b, ap-southeast2c. The !Select function then allows you to pick which values you want to use based on the index number you specified 0 is the first, and incremental from there.

    For more information about these functions, refer to link below :

    https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-select.html
    https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getavailabilityzones.html


-   Two private subnets with corresponding route tables
```
    ...
    SubnetPrivateSharedA:
        Type: "AWS::EC2::Subnet"
        Properties:
          AvailabilityZone: !Select [0, !GetAZs ]
          CidrBlock: !Ref psharedacidr
          MapPublicIpOnLaunch: false
          VpcId: !Ref VPC
    SubnetPrivateSharedB:
        Type: "AWS::EC2::Subnet"
        Properties:
          AvailabilityZone: !Select [1, !GetAZs ]
          CidrBlock: !Ref psharedbcidr
          MapPublicIpOnLaunch: false
          VpcId: !Ref VPC
    SubnetRouteTableAssociatePrivateA:
        Type: "AWS::EC2::SubnetRouteTableAssociation"
        Properties:
          RouteTableId: !Ref RouteTablePrivateA
          SubnetId: !Ref SubnetPrivateSharedA
    SubnetRouteTableAssociatePrivateB:
        Type: "AWS::EC2::SubnetRouteTableAssociation"
        Properties:
          RouteTableId: !Ref RouteTablePrivateB
          SubnetId: !Ref SubnetPrivateSharedB
    RouteTablePrivateA:
        Type: "AWS::EC2::RouteTable"
        Properties:
          VpcId: !Ref VPC
    RouteTablePrivateB:
        Type: "AWS::EC2::RouteTable"
        Properties:
          VpcId: !Ref VPC
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
    ...
```


-   Two Elastic IP addresses
    ```
    ...
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
    ...
    ```

    **Note:**

    In this section you should see the use of `DependsOn` under `EIPNatGWA` and `EIPNatGWB` resource. If you are creating relationship between resources using !GetAtt or !Ref, CloudFormation will handle the order of deployment orchestration for the resource. e.g: If you create and EC2 instance resource and it has security group value !Ref another Security Group resoruce, CloudFormation will create the Security Group first before creating EC2 instance. Now in the case where you don't have the !Ref values in the resource, but you would like to control the order of resources to be deployed, you can then use the `DependsOn` section under resource to define the order relationship. In this case `EIPNatGWA` and `EIPNatGWB` won't be created before `GatewayAttach` is successfully created.


-   Two NAT Gateways
    ```
    ...
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
    ...
    ```
    **Note:**

    In this section you should see a use case of Intrinsic Function !GetAtt `!GetAtt EIPNatGWA.AllocationId`. 
    !GetAtt function allows you reference additional attribute values of the resource that was launched. These are values outside what's been provided by !Ref. Every resource will have different attribute values available. Therefore to best use this, it's recommended to always refer to the documentation of the related resource type under `ReturnValues` `Fn::GetAtt`. In this example the !GetAtt function is referencing the Allocation Id value of Elastic IP resource called EIPNatGWA. 
    
    https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-eip.html
