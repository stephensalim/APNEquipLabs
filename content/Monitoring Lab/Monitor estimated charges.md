---
title: "8. Monitor Estimated Charges"
weight: 8
draft: false
---

In this scenario, you create an Amazon CloudWatch alarm that will
monitor your estimated Amazon Web Services (AWS) charges. When you
enable the monitoring of estimated charges for your AWS account, the
estimated charges are calculated and sent several times daily to
CloudWatch as metric data that is stored for 14 days. Billing metric
data is stored in the US East (N. Virginia) Region and represents
worldwide charges. This data includes the estimated charges for every
service in AWS that you use, as well as the estimated overall total of
your AWS charges. You can choose to receive alerts by email when charges
have exceeded a certain threshold. These alerts are triggered by
CloudWatch and messages are sent using Amazon Simple Notification
Service (Amazon SNS).

**Step 1: Enable Monitoring of Your Estimated Charges**

Before you can create an alarm on your estimated charges, you must
enable monitoring of your estimated AWS charges, which creates metric
data that you can use to create a billing alarm. It takes about 15
minutes before you can view billing data and create alarms. After you
enable billing metrics you cannot disable the collection of data, but
you can delete any alarms you have created. You must be signed in as the
account owner (the "root user") to enable billing alerts for your AWS
account.


**To enable monitoring of your estimated charges**

1.  **Open the Billing and Cost Management console
    at [https://console.aws.amazon.com/billing/home?\#](https://console.aws.amazon.com/billing/home?#/).**

2.  **In the spaces provided, enter your user name and password, and
    then click Sign in using our secure server.**

3.  **In the navigation pane, click Preferences, and then select
    the Receive Billing Alerts check box.**
    
    ![](/Detailed Instructions/Monitoring Lab/images/image19.jpg)

**Step 2: Create a Billing Alarm**

After you've enabled monitoring of your estimated AWS charges, you
can create a billing alarm in the Amazon CloudWatch console. In this
scenario, you'll create an alarm that will send an email message when
your estimated charges for AWS exceed 200 dollars. When you enable the
monitoring of your estimated charges for the first time, it takes about
15 minutes before you can view billing data and set billing alarms.

**To create a billing alarm**

1.  **Open the CloudWatch console
    at <https://console.aws.amazon.com/cloudwatch/>.**

2.  **If necessary, change the region to US East (N. Virginia). Billing
    metric data is stored in the US East (N. Virginia) Region and
    represent worldwide charges. For more information, see [Regions and
    Endpoints](http://docs.aws.amazon.com/general/latest/gr/rande.html).**

3.  **In the navigation pane, click Alarms, and then in the Alarms pane,
    click Create Alarm.**

4.  **In the CloudWatch Metrics by Category pane, under Billing Metrics,
    click Total Estimated Charge.**
    
    ![](/Detailed Instructions/Monitoring Lab/images/image20.jpg)

5.  Under **Billing \> Total Estimated Charge**, select
    the **EstimatedCharges** metric to view a graph of billing data in
    the lower pane.
    
    ![](/Detailed Instructions/Monitoring Lab/images/image21.jpg)

6.  Click **Next**, and then in the **Alarm Threshold** pane, in
    the **Name** box, type a unique, friendly name for the alarm (for
    example, My Estimated Charges).
    
    ![](/Detailed Instructions/Monitoring Lab/images/image22.jpg)

7.  In the Description box, enter a description for the alarm (for
    example, Estimated Monthly Charges).

8.  Under Whenever charges for, in the is drop-down list,
    select >= (greater than or equal to), and then in the USD box, set
    the monetary amount (for example, 200) that must be exceeded to
    trigger the alarm and send an email.

	**Note: Under Alarm Preview, in the Estimated Monthly
	Charges thumbnail graph, you can see an estimate of your charges that
	you can use to set an appropriate threshold for the alarm.**

9.  Under Actions, click Notification, and then in the Whenever this
    alarm drop-down menu, click State is ALARM.

10. In the Send notification to box, select an existing Amazon SNS
    topic.

	To create a new Amazon SNS topic, click Create topic, and then in the Send notification to box, enter a name for the new Amazon SNS topic (for example., CFO), and in the Email list box, enter the email address (for example, john.stiles@example.com) where email notifications should be sent.

	**Note: If you create a new Amazon SNS topic, the email account associated with the topic will receive a subscription confirmation email. You must confirm the subscription in order to receive future email notifications when the alarm is triggered.**
	
	![](/Detailed Instructions/Monitoring Lab/images/image23.jpg)
	
11. Click **Create Alarm**.

	**Important: If you added an email address to the list of recipients
	or created a new topic, Amazon SNS sends a subscription confirmation
	email to each new address shortly after you create an alarm. Remember
	to click the link contained in that message, which confirms your
	subscription. Alert notifications are only sent to confirmed
	addresses.**

12. To view your billing alarm in the CloudWatch console, in the
    navigation pane, under Alarms, click Billing.
    
**Step 3: Check Alarm Status**

**Now, check the status of the billing alarm that you just created.**
**To check alarm status using the CloudWatch console**

1.  Sign in to the AWS Management Console and open the CloudWatch
    console at<https://console.aws.amazon.com/cloudwatch/>.

2.  If necessary, change the region to US East (N. Virginia). Billing
    metric data is stored in the US East (N. Virginia) Region and
    represent worldwide charges. For more information, see [Regions and
    Endpoints](http://docs.aws.amazon.com/general/latest/gr/rande.html).

3.  In the navigation pane, under Alarms, click Billing.

**Step 4: Edit a Billing Alarm**

Let\'s say that you want to increase the amount money you spend with AWS
each month to \$400. You can edit your existing billing alarm and
increase the dollar amount that must be exceeded before the alarm is
triggered.

**To edit a billing alarm using the CloudWatch console**

1.  Sign in to the AWS Management Console and open the CloudWatch
    console at <https://console.aws.amazon.com/cloudwatch/>.

2.  If necessary, change the region to US East (N. Virginia). Billing
    metric data is stored in the US East (N. Virginia) Region and
    represent worldwide charges. For more information, see [Regions and
    Endpoints](http://docs.aws.amazon.com/general/latest/gr/rande.html).

3.  In the navigation pane, under **Alarms**, click **Billing**.

4.  In the list of alarms, select the check box next to the alarm you
    want to change, and then click **Modify**.

5.  Under **Alarm Threshold**, in the **USD** box, set the monetary
    amount (for example, 400) that must be exceeded to trigger the alarm
    and send an email, and then click **Save Changes**.

**Step 5: Delete a Billing Alarm**

Now that you have enabled billing, and you have created and edited your
first billing alarm, you can delete the billing alarm if you no longer
need it.

**To delete a billing alarm using the CloudWatch console**

1.  Sign in to the AWS Management Console and open the CloudWatch
    console at <https://console.aws.amazon.com/cloudwatch/>.

2.  If necessary, change the region to US East (N. Virginia). Billing
    metric data is stored in the US East (N. Virginia) Region and
    represent worldwide charges. For more information, see [Regions and
    Endpoints](http://docs.aws.amazon.com/general/latest/gr/rande.html).

3.  In the navigation pane, under **Alarms**, click **Billing**.

4.  In the list of alarms, select the check box next to the alarm you
    want to delete, and then click **Delete**.

5.  In the **Delete Alarms** dialog box, click **Yes, Delete**.