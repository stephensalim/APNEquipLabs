---
title: "8. Test Assesment"
weight: 8
draft: false
---

As I mentioned before, the main part of our solution is done !
In this section we will test the execution of our solution by passing in manual invocation to the `StartContinuousAssessment` Lambda Function.
In this test, you trigger a security assessment and monitor it. 

1. **To start golden AMI vulnerability assessments:**

    1.  Sign in to the [AWS Management Console](https://console.aws.amazon.com/console/home) and choose **Lambda** in the **Services **menu.
    2.  Choose **Functions**. In the **Functions** pane, choose the **StartContinuousAssessment** function.
    3.  Choose the **Select a test event** drop-down, and choose **Configure test events**.
    4.  On the **Configure test event** page, choose **Create new test event** and specify the event name as test.
    5.  Paste the following JSON in the editor box.

        <div class="hide-language">

            {
            "AMIsParamName": "ContinuousAssessmentInput"
            }

        </div>

    6.  Choose **Create**. Choose **Test**.

    The `StartContinuousAssessment` function runs for approximately five minutes. 
    
    while you wait, here's the explanation on what occurs in the background.

    ![](/AMI Inspector Lab/images/Testing01.png)

    1. What's happening is `StartContinuousAssessment` Lambda function will pull down the metadata information you created in this Lab. This metadata will contain the `InstanceType`, `Ami-Id`, and `UserData` information that the function needs to execute the next activity.

    2. Next, it will use the `Ami-Id` to launch an EC2 Instance with the `Instance Type` you specified, and put in the `UserData` information. It reads the JSON parameter stored in the [AWS Systems Manager](http://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html) (Systems Manager) Parameter Store. As you know, this JSON parameter contains the following metadata for each golden AMI:
            
        * `InstanceType` – A valid instance-type for launching an EC2 instance of the golden AMI.
        * `Ami-Id` – The ID of the golden AMI.
        * `UserData` – An operating system–compatible [user-data script](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) for installing the [Amazon Inspector agent](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_agents.html).

    3. When each instance starts, it installs the Amazon Inspector agent by using the user-data script provided in the JSON. The Lambda function then copies each golden AMI’s [tags](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html) (you will assign custom metadata in the form of tags to each golden AMI when you set up the solution) to the corresponding EC2 instance. 
    
        The function also adds a tag with the `key` of `continuous-assessment-instance` and `value` as `true`. This tag identifies EC2 instances that require the security assessments. 
    
        The Lambda function copies the AMI’s tags to the instance (and later, to the security findings found for the instance) to help you identify the golden AMIs for each security finding. After you analyze security findings, you can patch your golden AMIs.

    4. The Function will wait for the instance to launch and execute Amazon Inspector to inspect the EC2 instance. The first time the `StartContinuousAssessment` function runs, it creates:
        * An [Amazon Inspector assessment target](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_applications.html): The target identifies EC2 instances to assess by using the `continuous-assessment-instance` tag.
        * An [Amazon Inspector assessment template](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_assessments.html#inspector-assessment-templates): The template contains a reference to the Amazon Inspector assessment target created in the preceding step and the following AWS managed rules packages to evaluate:
        *   [Common Vulnerabilities and Exposures](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_cves.html) (CVEs)
        *   [Center for Internet Security (CIS) Benchmarks](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_cis.html)
        *   [AWS Security Best Practices](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_security-best-practices.html)
        *   [Runtime Behavior Analysis](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_runtime-behavior-analysis.html)

        For subsequent assessments, the `StartContinuousAssessment` function reuses the target and the template created during the first run of `StartContinuousAssessment` function.

        ---

        **Note:** 
        
        Amazon Inspector can start an assessment only after it finds at least one running Amazon Inspector agent. To allow EC2 instances to boot and the Amazon inspector agent to start, the Lambda function waits four minutes. Because the assessment runs for approximately one hour and boot time for EC2 instances typically takes a few minutes, all Amazon Inspector agents start before the assessment ends.

        ---

        Once the EC2 instance is successfully launched you should see below message in your Lambda execution indicating that the process is complete.

        ![Message showing the function has run successfully](https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2017/12/15/KW_1_1217.png "Message showing the function has run successfully")

        The next thing that will happen are below :

        ![](/AMI Inspector Lab/images/Testing02.png)

    5. After the Lambda function completes the assessment, Amazon Inspector publishes an assessment-completion notification message to an [Amazon SNS](https://aws.amazon.com/sns/) topic called `ContinuousAssessmentCompleteTopic`. SNS uses _topics_, which are communication channels for sending messages and subscribing to notifications. This includes the `AnalyzeInspectorFindings` and the email address you specified.

        Amazon Inspector agents collect behavior and configuration data, and pass it to Amazon Inspector. Amazon Inspector analyzes the data and generates [Amazon Inspector findings](http://docs.aws.amazon.com/inspector/latest/userguide/inspector_findings.html), which are possible security findings you may need to address.

        You can open Amazon Inspector and monitor the progress of the assessment you can do below :

        * Sign in to the [AWS Management Console](https://console.aws.amazon.com/console/home) and navigate to the [Amazon Inspector console](https://console.aws.amazon.com/inspector/).
        * On **Dashboard** under **Recent Assessment Runs**, you will see an entry with the status, **Collecting Data**. This status indicates that Amazon Inspector agents are collecting data from instances running your golden AMIs. The agents collect data for an hour and then Amazon Inspector analyzes the collected data.

        ![](/AMI Inspector Lab/images/InspectorCollectingData.png)
    
        * After Amazon Inspector completes the assessment, the status in the console changes to **Analysis complete**. Amazon Inspector then publishes an SNS message that triggers the `AnalyzeInspectorFindings` Lambda function. When `AnalyzeInspectorFindings` publishes results, you will receive an email containing consolidated assessment results. You also will be able to see the findings.

        * To see the findings in Amazon Inspector’s **Findings** section:

            * Sign in to the [AWS Management Console](https://console.aws.amazon.com/console/home) and navigate to the [Amazon Inspector console](https://console.aws.amazon.com/inspector/home).
            * In the navigation pane, choose **Assessment Runs**. In the table on the **Amazon Inspector – Assessment Runs** page**,** choose the findings of the latest assessment run.
            * Choose the settings icon and choose the appropriate tags to see the details of findings, as shown in the following screenshot. The findings also contain information about how you can address each underlying vulnerability. 

            ![Screenshot showing details of findings](https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2017/12/15/KW_2_1217.png "Screenshot showing details of findings")

    6. Along with the above,`AnalyzeInspectorFindings` will do the following:

        * Associates the tags of each EC2 instance with security findings found for that EC2 instance. This enables you to identify the security findings using the `app-name` tag you specified for your golden AMIs. You can use the information provided in the findings to patch your golden AMIs.
        * Terminates all instances associated with the `continuous-assessment-instance=true` tag.
        * Aggregates the number of findings found for each EC2 instance by severity and then publishes a consolidated result to an SNS topic called `ContinuousAssessmentResultsTopic`.

Now that We have succesfully Test our Solution, we are ready to create the schedule Task for our stack