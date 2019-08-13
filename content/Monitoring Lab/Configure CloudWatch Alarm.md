---
title: "Configure CloudWatch Alarm"
weight: 4
draft: true
---

1.  In the EC2 Console, click the checkbox next to your server name to
    view details about this EC2 instance. Click the **Monitoring** tab
    and then click **Enable Detailed Monitoring** to provide monitoring
    data at a 1 minute interval vs. the default of 5 minutes.
    
    ![](/Monitoring Lab/images/image13.jpg)

2.  Click the **Description** tab and copy your "Instance ID" to the
    clipboard or other location such as notepad.

3.  Click the **Monitoring** tab and click **Create Alarm**.

    ![](/Monitoring Lab/images/image14.jpg)
    
4.  To the right of the "Send a notification to:" drop down, select the
    SNS Topic you created in the previous step. In the "Whenever:"
    field, set the **Average** of **CPU Utilization** to ">=" 60%. In
    the "For at least:" field, set the "Consecutive periods to **1
    Minute**. Add your name to the "Name of alarm:" field and click
    **Create Alarm**. Note: If 1 minute period is not an option, click
    the **Description** tab and then click back to the **Monitoring**
    tab and proceed to set up the Alarm.

	![](/Monitoring Lab/images/image15.jpg)

5.  In the top left area of the AWS Console select **Services >
    CloudWatch**.

6.  Click Alarms in the left pane of the Console and check the State of
    your Alarm.

	![](/Monitoring Lab/images/image16.jpg)

7.  In the CloudWatch Console select **Metrics** in the left pane.
    Select the **All Metrics** tab and paste your Instance ID into the
    filter. Add an additional filter "cpu". Select **CPUUtilization**
    metric. Select the **Graphed metrics** tab and change the **Period**
    to 1 Minute. Change the graph interval to a custom value of 30m and
    select Auto refresh of 1min.
    
    ![](/Monitoring Lab/images/image17.jpg)

8.  After 5 minutes, the stress tool will begin to simulate CPU workload
    and trigger the Alarm once the threshold is reached. You can view
    the Alarm state in the CloudWatch console under **Alarms**. If you
    setup an email notification you will receive an email alert when the
    Alarm is triggered.
    
    ![](/Monitoring Lab/images/image18.jpg)
    

	**Great Job! You have successfully configured a CloudWatch Alarm!!**

