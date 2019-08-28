---
title: "7. Monitor EC2 Tag"
weight: 7
draft: false
---

In this scenario, you create an AWS Config rule, that will monitor your
EC2 instance resources in the region for a tag named "CostCenter" and
value "1000" to signify that the cost of this instance is being tracked.
If the EC2 instance does not have CostCenter tag, You can then set up
reconfig-mediation using AWS-StopEC2Instance AWS Systems Manager Automation.

1.  From the AWS console click **Services** > **Config**.

    If this is the first time you enabled AWS Config, your console will
    direct you to the "Get Started Page".

2.  Click on **Get started** button.

	![](/Monitoring Lab/images/config-media-image13.png)

3.  Tick on the **Record all resources supported in this region**.

	![](/Monitoring Lab/images/config-media-image14.png)

4.  Select **Create a bucket**.

5.  Entere a unique bucket name the **Bucket name** field.

	![](/Monitoring Lab/images/config-media-image15.png)

6.  Select **Use an existing AWS Config service-linked role**

	![](/Monitoring Lab/images/config-media-image16.png)

7.  Click on **Next**

8.  Click on **Skip** for now, and click on **Confirm\
    **This will prepare the AWS config resource recording ready for you
    to deploy the required-tags managed rule.

    ![](/Monitoring Lab/images/config-media-image17.png)
    
9.  From this point, you should be able to see your AWS Config menu on
    the left hand side of the console.

10. Click on **Rules > Add rule**

11. Search for **required-tags** and click on the managed rule.

    ![](/Monitoring Lab/images/config-media-image19.png)

12. On the **Trigger** section, remove Resources except for
    **EC2:Instance** to only allow this rule to check for changes on
    EC2:Instances

13. On the **Rule parameters** ensure that CostCenter is the value for
    **tag1Key**

    ![](/Monitoring Lab/images/config-media-image20.png)

14. Under **Choose reconfig-mediation action** select **AWS-StopEC2Instance**
    for the Reconfig-mediation action.

15. Select **InstanceId** for the **Resource ID parameter** and click
    **Save**

    This will connect your AWS Config rule with AWS Systems Manager
    Automation document to execute the pre-defined action to stop the
    EC2 instance. For more information about AWS System Manager
    Automation document please refer to this
    [page](https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-documents.html)

16. AWS config should now evaluate the the instances in your region.\
    Once it has completed it's evaluation, you should now see the EC2
    instance you launched in previous steps as being, Noncompliant.

    ![](/Monitoring Lab/images/config-media-image21.png)

17. Optionally, you can select the instance and click on **Reconfig-mediate**
    to execute the pre-defined reconfig-mediation action
    **AWS-StopEC2Instance.** You can do so and watch the EC2 instance
    being Stopped.

18. You can now go to the EC2 instance console in question, and add
    **CostCenter** Tag with any value in it. ( To edit the tag of ec2
    instance follow this
    [guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html#adding-or-deleting-tags)
    ).

19. Go back to the AWS Config console, click on **Rules >**
    **required-tags** Rule name.

20. Click on **Re-evaluate**

    You should now see the config running it's re-evaluation.

21. Once the evaluation complete, locate the instance in the Compliance
    list.

22. Click on **Compliance status** and change to **Compliant**.

23. Click on the EC2 instance resource and click on Compliance timeline
    to see the timeline of the changes on that EC2 instance.

    ![](/Monitoring Lab/images/config-media-image22.png)
  