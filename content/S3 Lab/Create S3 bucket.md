---
title: "1. Create S3 Bucket"
weight: 1
draft: false
---

Every object in Amazon S3 is stored in a bucket. Before you can store
data in Amazon S3 you must create a bucket.

***Note:** You are not charged for creating a bucket; you are only
charged for storing objects in the bucket and for transferring objects
in and out of the bucket.*

1.  Sign into the AWS Management Console and open the Amazon S3 console
    at
    [[https://console.aws.amazon.com/s3]{.underline}](https://console.aws.amazon.com/s3).

2.  Click **Create Bucket**. The **Create a Bucket** dialog box appears.

	![](/Detailed Instructions/S3 Lab/images/image3.png)

3.  Enter a bucket name in the **Bucket Name** field. The bucket name
    you choose must be unique across all existing bucket names in Amazon
    S3. One way to do that is to prefix your bucket names with your
    organisation\'s name.  

	Bucket names must comply with the following requirements.
	
	Bucket names:
	
	-   Can contain lowercase letters, numbers, periods (.) and dashes (-)
	-   Must start with a number or letter
	-   Must be between 3 and 255 characters long
	-   Must not be formatted as an IP address (e.g., 265.255.5.4)
	
	**Note:** There might be additional restrictions on bucket names based
	on the region your bucket is in or how you intend to access the object.
	Once you create a bucket, you cannot change its name. In addition, the
	bucket name is visible in the URL that points to the objects stored in
	the bucket. Make sure the bucket name you choose is appropriate.

4.  In the **Region** drop-down list box, select a region. Click
    **Next**.

5.  Under **Tags**, in the **Key** box, type **Purpose**, and in the
    **Value** box, enter **Immersion Day**.

6.  Have a good look at the other options, but leave them as default for
    this lab. Click **Next**
    twice.
    
    ![](/Detailed Instructions/S3 Lab/images/image4.png)

7.  Finally, review your settings and click **Create bucket**. When
    Amazon S3 successfully creates your bucket, the console displays
    your empty bucket in the **Buckets** panel.

	![](/Detailed Instructions/S3 Lab/images/image5.png)

**Well done - you've created your first bucket in Amazon S3!**

