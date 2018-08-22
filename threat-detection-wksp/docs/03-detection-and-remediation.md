# Module 3: Detection and Remediation

Unfortunately, due to a misconfiguration in your environment, a hacker has been able to gain access to your webserver. Now, with the intruder in your environment you’re getting alerts from the security tools you’ve put in place indicating nefarious activity. These alerts include communication with known malicious IP addresses, account reconnaissance, changes to S3 policies, and disabling security configurations. You must identify exactly what activity the intruder has performed and how they did it so you can block the intruder’s access, remediate the vulnerabilities, and restore the configuration to its proper state.

### Agenda

Detect, respond & remediate – 30 min

## Find out what's happening!

You’ve received the first alerts from GuardDuty. Now what? Since the alert came from GuardDuty, we will check there first.

### Check GuardDuty findings

1.  Go to the [Amazon GuardDuty](https://us-west-2.console.aws.amazon.com/guardduty/home?region=us-west-2) console.
2.  In the navigation pane, click on **Findings**.  You should see all the findings below:
    ![GuardDuty Findings](../images/03-gd-findings.png)

    > Don't panic if all of these findings fail to show up at first. The findings generated will take at least 20 minutes to show up in the GuardDuty console.
3.  The high severity ![High Severity](../images/03-high-severity.png) **UnauthorizedAccess:EC2/SSHBruteForce** finding is a result of the activity going on in the background to simulate the attack and can be ignored. You can archive the finding if you choose with the steps below:
    * Click on the check box next to the "**UnauthorizedAccess:EC2/SSHBruteForce**" **Finding**.
    * Click on **Actions**.
    * Select **Archive**.  

      > If you're interested in seeing all of your findings (current and archived) you can click on the filter icon to the left of *Add filter criteria* to toggle them in the console.
    * After archiving you should have four findings that are associated with this workshop.
4.  Now let's examine the low severity ![Low Severity](../images/03-low-severity.png) **UnauthorizedAccess:EC2/SSHBruteForce** finding since this was one of the first findings to show up. 
    * Click on the **Finding**.
    * Review the **Resource affected** and other sections in the window that popped open on the right.
    * Copy down the **GuardDuty Finding ID** and the **Instance ID**.

      > Was the brute force attack successful?

### Check Inspector assessment and CloudWatch logs

Following security design best practices you already setup your instances to send certain logs to CloudWatch. You’ve also setup a CloudWatch event rule that runs an [AWS Inspector](https://aws.amazon.com/inspector/) scan when GuardDuty detects a particular attack. Let’s look at the Inspector findings to see if the SSH configuration adheres to best practices to determine what the risk is for the brute force attack. We will then examine the CloudWatch logs.

1.  Go to [Amazon Inspector](https://us-west-2.console.aws.amazon.com/inspector/home?region=us-west-2) in the Amazon Console.
2.  Click **Findings** on the left navigation.
    
    > If you do not see any findings after awhile, there may have been an issue with your Inspector agent.  Click on **Assessment Templates**, check the template that starts with **threat-detection-wksp**, and click **Run**.  Please allow **15 minutes** for the scan to complete.  You can also look in **Assessment runs** and check the **status**. Feel free to continue through this module and check the results later on. 

3.  Filter down the findings by typing in **Password**.
4.  Review the findings.
    ![Inspector Findings](../images/03-inspector-findings.png)

    > Which Inspector [rule packages](https://docs.aws.amazon.com/inspector/latest/userguide/inspector_rule-packages.html) were used for this scan?

Based on the findings you see that password authentication is configured on the instance with no password complexity restrictions which means the instance is more susceptible to a SSH brute force attack. Now that we know that let’s look at the CloudWatch logs and create a metric to see if there are any successful SSH logins (to finally answer the question of whether the SSH brute force attack was successful.)

5.  Go to [CloudWatch logs](https://us-west-2.console.aws.amazon.com/cloudwatch/home?region=us-west-2#logs:).
6.  Click on the log group **/threat-detection-wksp/var/log/secure**
7.  If you have multiple log streams, filter using the Instance ID you copied earlier and click on the stream.
8.  Within the **Filter Events** text-box put the following Filter Pattern: 

    ```
    [Mon, day, timestamp, ip, id, msg1= Invalid, msg2 = user, ...]
    ```

    > Do you see any failed (invalid user) attempts to log into the instance?
9.  Now replace the Filter with one for successful attempts:

    ```
    [Mon, day, timestamp, ip, id, msg1= Accepted, msg2 = password, ...]
    ```

    > Do you see any successful attempts to log into the instance?

    > Which linux user was compromised?


### Check the remaining GuardDuty findings

Now that you have verified that the brute force attack was successful and your instance was compromised, go back to the [Amazon GuardDuty](https://us-west-2.console.aws.amazon.com/guardduty/home?region=us-west-2) console and view the other findings.

> Does it look like you are early in identifying the attack (just intrusion), or has the intruder started performing malicious actions?

View the following GuardDuty findings and take a note of the resources involved:

* **Recon:IAMUser/MaliciousIPCaller.Custom**
* **UnauthorizedAccess:IAMUser/MaliciousIPCaller.Custom**
* **UnauthorizedAccess:EC2/MaliciousIPCaller.Custom**

You can see by these findings that the compromised instance is communicating with an IP address on your custom threat list and that API calls are being made from a host on your custom threat list. 

> The API calls are being made using AWS IAM temporary security credentials coming from an IAM Role for EC2. How could you determine this fact yourself?
 
### Check if sensitive data was involved

At this point you know how the attacker was able to get into your systems and a general idea of what they did. If you review the permissions associated with the IAM Role attached to the compromised instance you will notice that it has a very permissive policy as it relates to your S3 data bucket.  Verify what sort of sensitive data is in your bucket and take a closer at your Macie Alerts.

1.  Go to the [Amazon Macie](https://mt.us-west-2.macie.aws.amazon.com/) console.
2.  Look through the latest alerts.

    > Do you see any critical alerts?  

Next lets verify what sort of sensitive data exists in that bucket.

3.  Click **Dashboard** on the left navigation.  You should see the following data classifications:
    ![Macie Classification](../images/03-macie-data.png)

    > You can slide the risk slider to filter data classifications based on risk levels.

4.  Above the risk slider, click the **S3 Public Objects and Buckets** icon. 
    ![Public Objects Button](../images/03-macie-public-objects-button.png)

5.  Click the magnifying glass to the left of the bucket name listed.
6. Check if any of the data in the bucket is considered a high risk

	> Look for the **Object PII priority** field and **Object risk level** field
	
6.  Verify if any of the data is unencrypted 

    > Look for the **Object encryption** field. (Does a portion of the blue bar indicate that encryption is set to none?)

At this point you have identified that there is a bucket with high risk data that has open public read permissions and that certain objects in that bucket are not encrypted.  When you first configured the environment you enabled default encryption for the bucket so this could be an indicator that someone has disabled it. 

Since you are already in the Macie service, create a new Basic Alert that will alert you in the future if default encryption is disabled on any of your buckets. 

7.  Click on **Settings** in the left navigation and then **Basic Alerts**.
8.  Click on **Add New**
9.  Create an alert with the following parameters:
    * **Alert title**: *Encryption Removed*
    * **Description**: *Evidence of encryption being removed from a bucket*
    * **Category**: *Data Compliance*
    * **Query**: *eventNameErrorCode.key:DeleteBucketEncryption*

      > Macie allows you to create queries using the [Apache Lucene Query Parser Syntax](https://docs.aws.amazon.com/macie/latest/userguide/macie-research.html#macie-query).  The query created for this alert uses the Macie field name **eventNameErrorCode.key** which corresponds to the CloudTrail field name **eventName**.
    * **Index**: *CloudTrail Data*
    * **Severity**: *Critical*
    
    You can leave the other options at the default settings
10.  Click **Save**
11. In the list of alerts find the alert you just created and click on the magnifying glass to the right of the screen to run the alert. 
12. Review the alert details.

Next you need to track down if someone recently disabled default encryption and who did it.

13. Go to the [AWS CloudTrail](https://us-west-2.console.aws.amazon.com/cloudtrail/home?region=us-west-2) console
14. Click **Event History** on the left navigation.
15. Set the **Filter** to **Event name** and for the event name enter **DeleteBucketEncryption**. Set the **Time Range** to today.
16. Expand the latest event and click on **View Event** to see the details of the API call.

    > Which AWS IAM Role disabled default encryption on the S3 Bucket?

    > Is this the same AWS IAM Role seen in your GuardDuty findings?

## Stop and evaluate

So at this point you have identified a successful intrusion into your network and specific actions taken against your account. Let’s recap what those are:

* A SSH brute force attack against an internet accessible EC2 instance was successful.
* The EC2 instance communicated with a known malicious IP address (possibly indicating that malware was installed.)
* The IAM credentials for the instance were stolen, published to S3, and used to perform reconnaissance against the account.
* An S3 bucket was made public and encryption was removed - most likely for data exfiltration.

Now that you've identified the attacker’s actions you need to stop them from performing any additional activities, restore your systems to their previous configurations, and protect your resources so this can’t happen again. 

## Respond and remediate

Before we get ahead of ourselves, we must stop any further actions from taking place. This requires removing the foothold in our environment, revoking any active credentials or removing those credentials' capabilities, and blocking further actions by the attacker. 

### Verify your automated remediation

Based upon your existing work, you’ve implemented the first step by using the CloudWatch Event rule to trigger the Lambda function to update the NACL for the subnet that the instance is located in. Let’s look at what changed.

1.  Go to the [AWS Config](https://us-west-2.console.aws.amazon.com/config/home?region=us-west-2) console.
2.  You may initially see the following message (if you don't see the message just go on to the next step):
    ![Config Message](../images/03-config-message.png)

    Click the **Click Here** button to proceed with using Config without Config Rules

3.  Click **Resources** in the left navigation.
4.  We can search here to find configuration changes to the NACL. Select the radio button for **Tag** and enter **Name** for the **Tag key** and **threat-detection-wksp-compromised** for the **Tag value** (this is allowing us to search for the NACL with that name):
    ![Config Key Pair](../images/03-config-keypair.png)
6.  Click on the Network ACL ID in the **Config timeline** column.
7.  On the next screen you should see two **Change** events in the timeline. Click on **Change** for each one to see what occurred.
8.  Evaluate the NACL changes.

    > What does the new NACL rule do?

### Modify the security group

Now that the active session from the attacker has been stopped by the update to the NACL, you need to modify the Security Group associated with the instance to prevent the attacker or anyone else from coming from a different IP.

1.  Go to the [Amazon EC2](https://us-west-2.console.aws.amazon.com/ec2/v2/home?region=us-west-2) Console.

2.  Find the running instances with the name **threat-detection-wksp: Compromised Instance**.

3.  Under the **Description** tab, click on the Security Group for the compromised instance.

4.  View the rules under the **Inbound** tab.

5.  Click **Edit** and delete the inbound SSH rule. You've decided that all administration on EC2 Instances will be done through [AWS Systems Manager](https://aws.amazon.com/systems-manager/) so you no longer need this port open.

    > In your initial setup you already installed the SSM Agent on your EC2 Instance.
6. Click **Save**

### Revoke the IAM role active sessions and rotate credentials

Now that the attacker can’t SSH into the compromised instance, you need to revoke all active sessions for the IAM role associated with that instance.  

First verify what the current credentials are for the EC2 instance.   

1.  Go to [AWS Systems Manager](https://us-west-2.console.aws.amazon.com/systems-manager/managed-instances?region=us-west-2) console and click **Managed Instances** (found under the **Shared Resources** section on the left navigation).

    > You should see an instance named **threat-detection-wksp: Compromised Instance** with a **Ping status** of **Online**.
2.  To see the keys currently active on the instance, click on **Run Command** on the left navigation and then click **Run a Command**.
4.  For **Command document** select **AWS-RunShellScript** 
	> You may need to scroll through the list or just enter the document name in the search bar.
5.  Leave the **Document version** at the default. 
6.  Under **Command Parameters** type the following in **Commands**:

    ```
    curl http://169.254.169.254/latest/meta-data/iam/security-credentials/threat-detection-wksp-compromised-ec2
    ```
7. Under **Targets** select **Manually selecting instances** and check the checkbox next to **threat-detection-wksp: Compromised Instance**.
8. Under **Output options** uncheck the checkbox labeled **Enable writing to an S3 bucket**.
9.  Leave all of the other options at their default, scroll to the end and click **Run**.
8.  Scroll down to the **Targets and outputs** section and click the  **Instance ID** (which should correspond to the instance ID of the compromised instance)
10. Expand **Step 1 - Output** and make note of the **AccessKeyID**, **SecretAccessKey**, and **Token**. (the command will take a minute or so to run)

	Now revoke the IAM Role session:

1.  Browse to the [AWS IAM](https://console.aws.amazon.com/iam/home?region=us-west-2) console.

2.  Click **Roles** and the **threat-detection-wksp-compromised-ec2** role (this is the role attached to the compromised instance).

3.  Click on the **Revoke sessions** tab.

4.  Click on **Revoke active sessions**.

5.  Click the acknowledgement **check box** and then click **Revoke active sessions**. 
	> What is the mechanism that is put in place by this step to actually prevent the use of the temp credentials issued by this role? 

#### Restart instance to rotate credentials

Now all active sessions for the compromised instance role have been invalidated.  This means the attacker can no longer use those credentials, but it also means that your application that use this role can't as well.  In order to ensure the availability of your application you need to refresh the credentials on the instance.  

To change the IAM credentials on the instance, you must Stop and Start the instance. A simple reboot will not change the keys.  Since you are using AWS Systems Manager for doing administration on your EC2 Instances you can use it to query the metadata to validate that the credentials were rotated.

16. In the [EC2 console](https://us-west-2.console.aws.amazon.com/ec2/v2/home?region=us-west-2#Instances:sort=instanceId) **Stop** the Instance named **threat-detection-wksp: Compromised Instance**.
17. Wait for the Instance State to say **Stopped** and then **Start** the instance.

	Lastly, you can need to query the metadata again to validate that the credentials were changed.

18. Repeat the first 10 Systems Manager steps to retrieve the credentials again.

    > Notice the keys are different.

    > If you want, try again after rebooting the instance. The keys will stay the same.

This is a good use case for auto-scaling groups and golden-image AMI’s, but that is out of scope for this workshop. Also out of scope is clearing the suspected malware.

### Limit S3 access

With the EC2 instance access scoped down and the IAM credentials revoked, we need to stop external access to the S3 bucket. Before we restore the previous configuration, we can quickly make the bucket only accessible from inside the VPC. Then we can re-enable encryption.

1.  First, check the configuration of the S3 endpoint in your environment by going to [Amazon VPC](https://us-west-2.console.aws.amazon.com/vpc/home?region=us-west-2) and clicking on **Endpoints** on the left navigation.
    * Copy the **Endpoint ID** (if there are multiple endpoints, look for the one with the VPC ID that ends in **threat-detection-wksp**)
2.  Click the **Policy** tab and review the policy statement. Notice that all access is allowed.
3.  Click the **Route Tables** tab to get the **Route Table ID** associated with this endpoint. You can then go to the VPC to see which subnets are associated with this route table and therefore the endpoint. 
    
    > Notice that routing to S3 is through the VPC endpoint

4.  Go to the [Amazon S3](https://s3.console.aws.amazon.com/s3/home?region=us-west-2) console.
5. Click the bucket that starts with **threat-detection-wksp-** and ends in **-data**.
6. Click on the **Permissions** tab.
7. Click **Bucket Policy**
8. Update the Bucket Policy with the following policy:

    ```
    {
      "Version": "2012-10-17",
      "Id": "Policy1415115909152",
      "Statement": [
        {
          "Sid": "Access-to-specific-VPCE-only",
          "Principal": "*",
          "Action": "s3:GetObject",
          "Effect": "Deny",
          "Resource": ["arn:aws:s3:::<BUCKETNAME>",
                       "arn:aws:s3:::<BUCKETNAME>/*"],
          "Condition": {
            "StringNotEquals": {
              "aws:sourceVpce": "<VPCENDPOINTID>"
            }
          }
        }
      ]
    }
    ```

    > Be sure to replace **<BUCKETNAME>** with the name of the bucket and **<VPCENDPOINTID>** with endpoint ID you copied down earlier.

9.  Click **Save**. Now regardless of the bucket’s Access Control List, if the request for **get object** isn’t coming from the VPC endpoint it will be denied. This also provides more security and better pricing for legitimate traffic. 
> Is there still a problem with the bucket permissions given the Access Control List that is still in place?
10. To quickly restore the previous configuration for this bucket, we start by going to the [AWS Config](https://us-west-2.console.aws.amazon.com/config/home?region=us-west-2) console.
11. Under **Resources** click on **S3 Bucket**.
12. Click on your Bucket (bucket name starts with **threat-detection-wksp-** and ends in **-data**).
13. Click on **Change** under the previous configuration in the slider at the top of the screen
    * Make note of the changes to Permissions
14. Click on **Manage Resource** in the top right. This will take you to the S3 console.
15. Click on the **Permissions** tab.
16. Remove Public Access based on Config change you just looked at.
17. Click on the **Properties** Tab.
18. Re-enable S3 Default AES-256 encryption.

With the configuration reestablished and some additional protections in place, you can focus on alerts and automated remediations in the event of another attack.

After you have remediated the incident and further hardened your environment, you can proceed to the next module.

> If you are going through this workshop in a classroom setting then the instructor should start the module 4 presentation soon.

### **[Module 4 - Review and Discussion](../docs/04-review-and-discussion.md)**