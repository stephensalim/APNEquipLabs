---
title: "2. Add Object to S3 Bucket"
weight: 2
draft: false
---

Now that you've created a bucket, you're ready to add an object to it.
An object can be any kind of file: a text file, a photo, a video and so
forth. When you add a file to Amazon S3, you have the option of
including metadata with the file and setting permissions to control
access to the file.

1.  In the Amazon S3 console, click the on the name of the bucket to
    which you want to upload an object (not the bucket icon itself,
    though), and then click **Upload** in the **Objects** tab. The
    **Upload** dialogue opens (its appearance may differ slightly
    between different browsers).

2.  Click **Add Files** to select a file to upload. A file selection
    dialog box opens.

3.  Select a small file to upload and click **Open**. The **Upload**
    dialogue shows the files and folders you\'ve selected to upload.

4.  Click **Upload**. You can watch the progress of the upload at the
    bottom of the screen. This appears as soon as the upload begins.

> **Good work - you\'ve added a file to your bucket!**

View an Object
==============

Now that you\'ve added an object to a bucket, you can open and view it
in a browser.

1.  In the Amazon S3 console, click the on the name of the object you
    want to open.

	**Note:** By default, your Amazon S3 buckets and objects are private.
	To view an object using a URL, for example,
	https://s3.amazonaws.com/Bucket/Object the object must be publicly
	readable. Otherwise, you will need to create signed URL that includes a
	signature with authentication information. You can optionally save the
	object locally.

2.  Click on the **Permissions** tab. In the **Manage public
    permissions** section, click the radio button next to **Everyone**.
    In the Everyone dialogue, under the **Object access** section, tick
    the **Read** box, and then click **Save.**

	![](/Detailed Instructions/S3 Lab/images/image6.png)

3.  Click on the **Overview** tab, and then click on the link to your
    object at the bottom of the screen to view the file using your
    browser.

	**Good job - you've retrieved your object from S3 via the web!**
