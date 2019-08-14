---
title: "Create Alarm For Security Log"
weight: 7
draft: false
---

In this section, we will create a filter to the /var/log/secure LogGroup to locate any failed SSH connection attempt.

1. Got to CloudWatch Console, by typing in **CloudWatch** in the search bar and and press enter.

2. On the left hand side of the menu, click on **Logs**.

3. Type in **/var/log/secure** in the filter field.

	![](/Security Lab/images/image25.png)

4. Select on the **/var/log/secure** log group, then click on **Create Metric Filter**
	
	![](/Security Lab/images/image28.png)

5. Enter `[Mon, day, timestamp, ip, id, status = Invalid, ...]` in the Filter pattern field, then click **Assign Metric**

	![](/Security Lab/images/image29.png) 

6. Enter `Invalid SSH attempt filter` in Filter Name.

7. Enter `SSH` in Metric dimension.

8. Enter `Invalid SSH sttempt` in Metric dimension, then click **Create Filter**

	![](/Security Lab/images/image30.png) 

9. You should now be able to see the filter being created as below, and click **Create Alarm**

    ![](/Security Lab/images/image31.png) 

10. Set `Sum` Statistic, and 10 seconds for `Period`.

11. Please refer to this picture for the rest of the settings, once you are done click **Next**

    ![](/Security Lab/images/image32.png) 

12. Configure the Alarm to send notification to the SNS topic you created before as per the image below, then click **Next**

    ![](/Security Lab/images/image33.png) 

13. Enter the unique Alarm name for easy identification, then click **Next**

14. Confirm the settings of the alarm and click **Create Alarm**

    ![](/Security Lab/images/image34a.png)
    ![](/Security Lab/images/image34b.png) 