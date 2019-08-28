---
title: "4. Enable Bucket Versioning"
weight: 4
draft: false
---

If you want to add new version of the object to the same bucket but want
to retain the old version, you can turn on bucket versioning.

1.  In the S3 Console, click on the link representing the bucket you
    created, and then select the **Properties** tab.

	![](/Detailed Instructions/S3 Lab/images/image7.png)

2.  Click on the **Versioning** tile, select the **Enable versioning**
    radio button, and then click **Save**.

3.  Choose an object that you are able to edit on your computer, and
    upload it using the steps from the **Add an Object to a Bucket**
    section above.

4.  Now open the original file on your computer and edit it, saving the
    updated version under the **same file name**.

5.  Upload this updated file to the S3 bucket in the same way as before.

6.  Now click on the object's link in the S3 bucket and click on the
    words **Latest version** (to the right of the object's name).

	![](/Detailed Instructions/S3 Lab/images/image8.png)

	This shows the different versions of the object in the bucket. You can
	click on the **download** icon and view the different versions of the
	object.
