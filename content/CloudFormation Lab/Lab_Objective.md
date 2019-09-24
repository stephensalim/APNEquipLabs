---
title: "3. Change Your Stack"
weight: 3
draft: false
---


Now you need to modify your template with following objectives:

**Add Parameter Constraint :**

-   Vpccidr

    -   Minimum length should be set to 9

    -   Maximum length should be set to 18

    -   Allowed pattern should be:
        `((\d{1,3})\.){3}\d{1,3}/\d{1,2}`

    -   Add a constraint description

-   Psharedacidr

    -   Minimum length should be set to 9

    -   Maximum length should be set to 18

    -   Allowed pattern should be:
        `((\d{1,3})\.){3}\d{1,3}/\d{1,2}`

    -   Add a constraint description

-   Psharedbcidr

    -   Minimum length should be set to 9

    -   Maximum length should be set to 18

    -   Allowed pattern should be:
        `((\d{1,3})\.){3}\d{1,3}/\d{1,2}`

    -   Add a constraint description

**Add delete policy constraint :**

-   Create a Deletion Policy for your S3 bucket to be Retained at
    deletion

**Add Outputs section to show value in Output tab:**

-   Vpc id

    -   Create a description of your output

    -   Reference your VPC as the value using !Ref

-   NATGWA

    -   Create a description of your output

    -   Reference your NAT gateway A as the value using !Ref

-   NATGWB

    -   Create a description of your output

    -   Reference your NAT gateway B as the value using !Ref

-   App bucket URL

    -   Create a description of your output

    -   Reference your S3 bucket URL as the value using !Ref

**Add export values in Outputs section for Cross-Stack Reference:**

-   Vpc id

    -   Export your vpcid Name as 'sharedinf-vpc'

-   App bucket URL

    -   Export your appbucketurl Name as 'sharedinf-appbucketurl'

References:
===========

**Parameters:**

<https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html>

**Intrinsic functions:**

<https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html>

**Outputs and Export:**

<https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/outputs-section-structure.html>

**Mappings:**

<https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/mappings-section-structure.html>

**Deletion policy:**

<https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-deletionpolicy.html>
