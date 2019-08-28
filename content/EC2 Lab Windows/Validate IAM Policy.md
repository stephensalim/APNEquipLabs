---
title: '5. Validate IAM Policy'
weight: 5
draft: false
---

In this task, you will attempt to create a Windows Server 2016 EC2
instance in the **us-east-1 region** (N.Virginia). You will perform the
following steps as the newly created "West_Admin" user.

1.  In a new browser session (Private browsing or from another maching)
    log in as **West_Admin**

2.  Ensure that you are working in the **us-east-1** (N.Virginia) region
    Navigate to EC2 Console and click "Launch Instance"

3.  In the search field type: Windows

4.  Hit enter and scroll through the Windows AMI's, select **Windows
    2016 Base**

5.  Click **Select**

6.  Take note of the error message, this is due to the region
    restriction policy you created earlier.
