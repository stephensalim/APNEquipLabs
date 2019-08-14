---
title: "S3 Lab"
weight: 4
draft: false
---

**Amazon Simple Storage Service (S3)** provides a simple web services
interface that can be used to store and retrieve any amount of data, at
any time, from anywhere on the web. This lab is designed to demonstrate
how to interact with S3 to store, view, move and delete objects.

This lab will walk you through the following:

-   Creating a bucket in S3
-   Adding an object to the S3 bucket
-   View the object in S3
-   Move the object in S3
-   Enable bucket versioning
-   Delete the object and the bucket in S3


**Challange**

1. Create a new S3 bucket
2. Upload an Object to your bucket
	* Can you access via Web?
3. Make object public
	* Verify access via Web
4. Create a second S3 bucket
5. Move object to new S3 bucket
6. Enable versioning
	* Investigate object updates and delete

**Additional challange**

1. Create a static WebSite 
2. Host it on your S3 bucket

**Hints:**

1. Apply the following bucket policy

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::YOURBACKETNAME/*"
        }
    ]
}
```

2.	Create a simple files
	* index.html
	* Error.html

3. Upload them to your S3 bucket
4. Set up Bucket Static Web Site Hosting

**Clean up your account**

1. Delete all objects
2. Delete all S3 buckets	

**Documentation References:**

* [https://docs.aws.amazon.com/AmazonS3/latest/gsg/CreatingABucket.html](https://docs.aws.amazon.com/AmazonS3/latest/gsg/CreatingABucket.html)
* [https://docs.aws.amazon.com/AmazonS3/latest/gsg/PuttingAnObjectInABucket.html](https://docs.aws.amazon.com/AmazonS3/latest/gsg/PuttingAnObjectInABucket.html)
* [https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html)
* [https://docs.aws.amazon.com/AmazonS3/latest/dev/ObjectVersioning.html](https://docs.aws.amazon.com/AmazonS3/latest/dev/ObjectVersioning.html)
* [https://docs.aws.amazon.com/AmazonS3/latest/dev/Versioning.html](https://docs.aws.amazon.com/AmazonS3/latest/dev/Versioning.html)
* [https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingBucket.html#bucket-config-options-intro](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingBucket.html#bucket-config-options-intro)
* [https://docs.aws.amazon.com/AmazonS3/latest/user-guide/add-bucket-policy.html](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/add-bucket-policy.html)
