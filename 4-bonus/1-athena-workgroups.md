# Athena Workgroups to Control Query Access and Costs (Optional)

Use workgroups to separate users, teams, applications, or workloads, to set limits on amount of data each query or the entire workgroup can process, and to track costs. Because workgroups act as resources, you can use resource-level identity-based policies to control access to a specific workgroup. You can also view query-related metrics in Amazon CloudWatch, control costs by configuring limits on the amount of data scanned, create thresholds, and trigger actions, such as Amazon SNS, when these thresholds are breached.
Workflow setup to separate workloads

For this lab, we will create two workgroups: `workgroupA` and `workgroupB`. Before creating the workgroups, you need to have users, appropriate IAM policies to assigned to each user and S3 buckets to store the query results. We will have two users: `business_analyst_user` and `workgroup_manager_user` created in IAM with different policies:

- The `business_analyst_user` will have access to `workgroupA` and query `sporting_event_info` view.
- The `workgroup_manager_user` will have access to both workgroups `workgroupA` and `workgroupB` for management purposes.

This has been created using Cloud Formation template for your convenience. You can click on the CloudFormation stack and navigate to “Resources” to understand the different resources that are created within your account.

![screenshot](img/1.png)

Now we will create workgroups.

1. Navigate to Athena Console and click on “Workgroup: primary”. The default workgroup provided for querying in Athena is “primary”.

    ![screenshot](img/2.png)

2. Click on “Create workgroup”.

    ![screenshot](img/3.png)

3. Provide the following:
    - Workgroup Name: “workgroupA”

    > [!NOTE]
    > Please ensure workgroup names are provided with no typos and same casing as otherwise future steps will fail

    - Description (optional):
        - “workgroupA for BusinessAnalystUser”
        - “workgroupB for workgroup manager user”
    - Query result location: you can find the corresponding S3 bucket name from Cloudformation output tab of student lab
        - For workgroupA, the s3 path would look something like: `s3://mod-XXX-s3bucketworkgroupa-YYY/`.
        - For workgroupB, provide S3 path as: `s3://mod-XXX-s3bucketworkgroupb-ZZZ/`.
    - For “Encrypt query results”, leave as default i.e. unchecked. You can check this if you want your query results to be encrypted.
    - Check the checkbox for “Metrics: Publish query metrics to AWS CloudWatch”.

    ![screenshot](img/4.png)

4. Provide the following:
    - Optionally, you can click on Override client-side settings. This will override the client-side settings and keep the defaults for query execution and storing results.
    - Tag your workgroup to analyze later with CloudWatch or perform any analytics on query execution and results.
        - For workgroupA: provide key:name, value:workgroupA
        - For workgroupB: Provide key:name, value:workgroupB
    - For “Requester Pays S3 buckets”, keep as default. This is Optional. Choose Enable queries on Requester Pays buckets in Amazon S3 if workgroup users will run queries on data stored in Amazon S3 buckets that are configured as Requester Pays. The account of the user running the query is charged for applicable data access and data transfer fees associated with the query.

5. Click on “Create workgroup”.

6. Follow the above procedure to create “workgroupB”.

## Explore the features of workgroups

1. Go to CloudFormation, open the Resources tab for the single stack on it. Note down Physical ID for “BusinessAnalystUser” (`mod-XXX-BusinessAnalystUser-YYY`) and bucket name “S3BucketWorkgroupA” (`s3://mod-XXX-s3bucketworkgroupa-YYY/`) and put them aside.

2. Note down 12 digit AWS account id. Follow steps here to find out account id - https://www.apn-portal.com/knowledgebase/articles/FAQ/Where-Can-I-Find-My-AWS-Account-ID

3. Next, Open AWS console log-in different browser, select IAM user and login with following credential:

    - AccountID
    - IAM User name (from the step above)
    - Password: `u6%+QF,)+4FG` (for security reasons, you will need to set up a new password after logging in)

4. From new BusinessAnalystUser user, navigate to Athena console. You will notice that you can see your workgroup designated as “workgroupA” and you can also view table: sporting_event_info as shown below:

    ![screenshot](img/6.png)

    If your workgroup is other than `workgroupA`, click on Workgroup:

    ![screenshot](img/7.png)

    Select `workgroupA` from the workgroup list and then click on “Switch workgroup”.

    ![screenshot](img/8.png)

5. If you see that your bucket is not setup with Athena to store the query results, as shown below: then proceed to setup the bucket.

    ![screenshot](img/9.png)

6. Setup the S3 bucket for storing the query results. Click on “Settings”.

    ![screenshot](img/10.png)

    Provide the S3 bucket location for workgroupA, copied and saved from the Resources tab of cloud formation template, as shown below. Then, click on Save.

    ![screenshot](img/11.png)

7. Back to Athena Query Editor, click on the three dots against “sporting_event_info” view and then click on “Preview”. You will be able to see query results. This shows that you as “business_analyst_user” has access to query the view “sporting_event_info” and store the query results in S3 bucket designated for workgroupA.

    ![screenshot](img/12.png)

1. Logged in as “business_analyst_user”, click on “Workgroup” and try switching to other workgroups which this user does not have access to. Select “workgroupB” and then click on “switch workgroup”.

    ![screenshot](img/13.png)

9. If you try running the query again, you will get the error “Access Denied” as shown below:

    ![screenshot](img/14.png)

    This means that we have achieved the user segregation for different workgroups as defined by the IAM policy and attached to that user. Any query executed and its results within a particular workgroup will be isolated to that workgroup.

10.	To view the query results, navigate to “Workgroup”, then select `workgroupA` and click on “View Details”.

    ![screenshot](img/15.png)

11.	You will be able to see the details, as shown below. Navigate to S3 bucket by clicking on the link and see the query results stored inside the “Unsaved” folder within the workgroupA bucket.

    ![screenshot](img/16.png)

12.	Now, login as `workgroup_manager_user`.
    - Account ID
    - IAM User Name (something of the form `mod-XXX-WorkgroupManagerUser-YYY`)
    - Password: `;@idC&q)hUk8` (for security reasons, you will need to set up a new password after logging in)

    This user has access to workgroupA and workgroupB for management purposes. Go to Athena and try to View Details of primary workgroup. You will not be able to access the primary workgroup because this user does not have access to it.

    ![screenshot](img/17.png)

    ![screenshot](img/18.png)

    Also note that this user does not have access to any tables or cannot run any queries. This is how we can isolate the responsibilities of different users within different workgroups by defining policies and attaching that to the user.

    ![screenshot](img/19.png)

    At any point of time, you can edit, delete and disable your workgroups as shown:
    - Select the workgroup and click on “View Details”.

        ![screenshot](img/20.png)

    - Click on “Edit Workgroup” to make changes, “Delete workgroup” to delete the entire workgroup and “Disable workgroup” to disable the workgroup and disable any queries to be run within that workgroup.

        ![screenshot](img/21.png)

        > [!NOTE]
        > Please Note: For lab purpose, we are attaching policies directly to users. For best practices, we recommend creating separate groups in IAM for different workgroups and then attaching policies for different workgroups to their respective groups in IAM.

## Managing Query Usage and Cost

Note that the following section of this lab is carried out under admin account and not the BusinessAnalystUser and WorkgroupManagerUser, so please log back to the account you first signed in.

As you enabled CloudWatch metrics for your workgroups, you will be able to see Metrics, by selecting the desired workgroup and click on “Metrics” as shown:

![screenshot](img/22.png)

Choose the metrics interval that Athena should use to fetch the query metrics from CloudWatch, or choose the refresh icon to refresh the displayed metrics.

![screenshot](img/23.png)

Let’s setup data usage controls which means setting up the threshold for the amount of data scanned. There are two types of data usage controls: per-query and per-workgroup.

Per-query data usage control will check the total amount of data scanned by per query within the workgroup and if the amount exceeds the threshold, the query will be cancelled automatically. Let’s setup per-query data usage for “primary workgroup”.

1. From Athena console, click on “Workgroup” and select “primary”. Click on “View Details”.

    ![screenshot](img/24.png)

1. Click on “Data usage controls”. In “Per query data usage control”, the default minimum limit is 10MB per query. We will select that default value. Also, note the default “Action” for per query data usage control. If the query exceeds the limit, it will be cancelled.

3. Click “Update”.

4. The per-query threshold has been set.

    ![screenshot](img/25.png)

5.	Navigate to query editor on Athena console. Run the following query:

    ```sql
    SELECT * FROM "ticketdata"."sporting_event_ticket_info";
    ```

1. This query scans more thatn 10 MB of data which is the threshold that we set. Thus,this query execution will be cancelled, as shown:

    ![screenshot](img/26.png)

For per-workgroup data usage control, you can configure the maximum amount of data scanned by all queries in the workgroup during a specific period. This is useful when you have few analytics reports to run, where you probably have a good idea of how long the process should take and the total amount of data that queries scan during this time. You only have a few reports to run, so you can expect them to run in a few minutes, only scanning a few megabytes of data.

1. On Athena console, click on “Workgroup” and Select “workgroupA”. Click on “View Details”.

    ![screenshot](img/27.png)

2. Click on “Data usage Controls” and scroll down to section “Workgroup data usage controls”. Click on “Create workgroup data usage control”.

    ![screenshot](img/28.png)

3. The select query on `sporting_event_info` returns more than 10KB of data. For this lab, we have only this view to query from. So, let’s set the threshold accordingly.
    - Set “Data Limits” to 10 KBs
    - Set “Time period” to 1 minute
    - Set “Action” as “Send a notification to”. Here, click on “Create SNS Topic”.
        - This will take you to SNS Console. Provide Topic Name as “workgroupA”.

        ![screenshot](img/29.png)

        - Click on “Next Step”, then Create.
        - Click on “Create Subscription”. We will subscribe to this topic with email address. Whenever the threshold is breached, we will get an email notification to the email address which is our subscriber.

        ![screenshot](img/30.png)

        - In Create Subscription, select “Protocol” as Email. In “Endpoint”, Provide email address. Then click on “Create subscription”.

        ![screenshot](img/31.png)

        - Verify your email for subscription to be validated.
        - Back to WorkgroupA workgroup data usage control, for “Action”, select “workgroupA” for the SNS topic. Click on “Create”.

        ![screenshot](img/32.png)

        - Once created, this control will be listed like this:

        ![screenshot](img/33.png)

        > [!NOTE]
        > if the SNS topic doesn´t show you will need to copy the arn of the topic created in the previous steps and add it here

4. Back to Athena Query Editor, run the following query, by logging in as Business Analyst User to the console and selecting “Workgroup: workgroupA”:

    ```sql
    SELECT * FROM "ticketdata"."sporting_event_info";
    ```

5. You will receive an email notification from AWS Notifications stating that workgroup data usage threshold has been breached, which will look something like this:

    ![screenshot](img/34.png)

6. You can also check CloudWatch Alarms and get more details on CloudWatch console:

    ![screenshot](img/35.png)

7. Alternatively, you can have AWS Lambda as the subscriber endpoint, so as soon as the threshold is breached, SNS will call the lambda function, which in turn will disable the workgroup and preventing from executing further queries within that workgroup. Feel free to explore multiple subscriber endpoints.

