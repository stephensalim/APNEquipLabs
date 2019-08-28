---
title: "10. Query Security Log"
weight: 10
draft: false
---

1. Go back to CloudWatch console.

2. Click on **Logs > Insights**.

3. Under the Select log group(s) section click and select **/var/log/secure**

    ![](/Detailed Instructions/Security Lab/images/image39.png)

4. Click **Run query** and watch the Logs populate as per screenshot below.

    ![](/Detailed Instructions/Security Lab/images/image40.png)

5. Now going through for these log and locate the invalid user might be a bit of a hassle, let's put in a filter to only see failed SSH attempts.

6. Update the query field with this query, and click on **Run Query**

    ```
    fields @timestamp, @message
    | filter @message like /invalid/
    | sort @timestamp desc
    | limit 20
    ```

7. Expand the log to find more details about this event.

    ![](/Detailed Instructions/Security Lab/images/image41.png)

**Congratulations you have now successfuly created a security alarm for your EC2 instance !!**