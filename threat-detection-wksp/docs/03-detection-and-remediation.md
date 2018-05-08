# Module 3: Detection and Remediation

Unfortunately, due to a misconfiguration in your environment, a hacker has been able to gain access to your webserver. Now, with the intruder in your environment you’re getting alerts from the Security tools you’ve put in place indicating nefarious activity. These alerts include changes to communicating with known malicious IP addresses, account reconnaissance, changes to S3 policies, and disabling security configurations. You must identify exactly what activity the intruder has performed and how so you can block the intruder’s access, remediate the vulnerabilities, and restore the configuration to its proper state.

## Find Out What's Happening!

You’ve received the first alerts from GuardDuty. Now what? Assuming you’re still logged into your account, let’s see where these findings are coming from.
Since the alert came from GuardDuty, we will check there first.

### Check GuardDuty findings

1. Go to [Amazon GuardDuty](https://us-west-2.console.aws.amazon.com/guardduty/home?region=us-west-2) in the Amazon Console.
2. In the navigation pane, click on **Findings**.  You should see all the findings below:
  ![GuardDuty Findings](../images/03-gd-findings.png)

  > Don't panic if you don't see all these. You may just have to wait a little bit longer

3. Do you see these four findings?
  *	Do you see any IP’s performing malicious actions?
  * IP from our custom threat list should show up. Where would you find that?
  * Do you see any inbound brute force attacks? Can you tell if it was successful?

> The SSH Brute Force Finding from an internal IP (High Severity) is a function of our environment and can be ignored.
Let’s evaluate the intrusion. First, was the brute force successful? 

### Check Instance Security Configuration and Logs

Following security design best practices you already setup your servers to log CloudWatch. You’ve also setup automated scanning of instances under attack using Inspector. Let’s look at Inspector to see if password authentication is enabled for the instance under attack.

4. Go to the Inspector Console
5. Click on Findings in the left navigation
  * There should be three findings of medium severity
6. Evaluate each finding to determine if the instance is configured to support password authentication over SSH
  * Write down the instance ID of the compromised instance. You will use this later

Now we’ve determined that the instance is susceptible to a brute force SSH attack. Let’s look at the CloudWatch logs and create a metric to see if there are any successful attempts.

7. Go to the CloudWatch logs
8. Choose Logs in the navigation pane, click on the log group “/threat-detection-wksp/var/log/secure”
9. Click on the latest log stream
10. Where it says Filter Events put the following Filter Pattern
  * [Mon, day, timestamp, ip, id, msg1= Invalid, msg2 = user, ...]
  * Do you see any failed attempts to log into the instance?
11. Now replace the Filter with one for successful attempts
  * For Filter Pattern enter: [Mon, day, timestamp, ip, id, msg1= Accepted, msg2 = password, ...]
  * Do you see any successful attempts to log into the instance from the same IP or usernames?
12. If there is a match then you know the SSH Brute Force attack was successful
13. Go back to the GuardDuty console
14. Does it look like we are early in the attack (just intrusion), or has the intruder started performing malicious actions?
  * Account Reconnaissance
  * Malicious IP Communication
 
At this point we know how the attacker was able to get into the system and what they did. But what about the data in the S3 bucket? Did they do anything to that? Have you received any alerts from Macie about data in your buckets?

15. Go to the Macie console
16. The alerts show up immediately
  * Were any actions taken in the last 30 minutes?
  * Did Macie find any sensitive data in our buckets we should be worried about?
17. Click on “Dashboard”
  * What types of files are at high risk (>5)?
  * Are access keys found in your bucket?
18. Click the icon under “Critical Assets” for “S3 Objects by PII”
  * Is there any PII in your bucket?

Now you know all of your PII should be encrypted, but what if the attacker removed that encryption? Rather than checking each file in S3, you can create a Macie alert to validate if encryption has been disabled.

19. Click on Settings and then Basic Alerts
20. Click on Add New
21. Create an alert with the following parameters:
  * Alert title: “Encryption Removed”
  * Description: “Evidence of encryption being removed from a bucket”
  * Category: “Data Compliance”
  * Query: eventNameErrorCode.key::DeleteBucketEncryption
  * Index: “CloudTrail Data”
  * Severity: “Critical”
  * You can leave the other options at the default settings
22. Click Save
23. In the list of alerts find the alert you just created and click on the magnifying glass to the right of the screen to run the alert. 
24. Review the alert details

We now see that a bucket has had encryption removed. In order to find out which bucket, let’s look at the CloudTrail logs we enabled.

25. Going back to the AWS Console go to the CloudTrail console
26. Check to see if the “DeleteBucketEncryption” Event shows up in the Recent Events on the Dashboard
27. Click on View Event to see the details of the call
  * Which bucket was encryption disabled on?
  * Why would it not be in the Dashboard (yet)?

Stop and Evaluate

So at this point we have identified a successful intrusion into our network and specific actions taken against our account. Let’s recap what those are:

1. Brute force SSH Attack against an internet accessible EC2 instance was successful
2. Malware was put on the EC2 instance and communicated with a known malicious IP address
3. The IAM credentials for the server were stolen, published to S3, and used to perform reconnaissance against the account
4. An S3 bucket was made public and encryption was removed - most likely for data exfiltration
Now that we’ve identified the attacker’s actions we need to stop them from performing any additional activities, restore our systems to their previous configurations, and protect our resources so this can’t happen again. For each of the following vulnerabilities, develop a remediation using AWS native policy and a DevSecOps approach to alerting and remediation during future attacks. (Answers are not shared with Attendees)
A.>	EC2 Brute Force attack:
a.	(Answers) Reduce access to only specific IP’s
i.	Update Security Groups to reduce Port 22 access to MyIP (Manual intervention)
b.	(Answers) Act upon alert on failed SSH attempts that blocks the IP address automatically (Lambda adds NACL)
c.	(Answers) Improve username/password combinations (Switch back to key pairs and Run Command)
B.>	Malware Infection:
a.	(Answers) Automate GuardDuty finding to NACL blocking (GD findings -> CloudWatch event -> Lambda -> NACL)
C.>	EC2 Credential Theft:
a.	(Answers) Rotate keys so the IAM credentials stop being useful (Stop and Start Instance, and TBD)
b.	(Answers) Change IAM credentials to least privilege (Demonstrate better policy)
D.>	S3 Bucket Policy
a.	(Answers) Reestablish access rules (Look at Config)
b.	(Answers) Change S3 bucket access to only S3 Endpoint (VPC Routing)
c.	(Answers) Automate unencrypted bucket findings and lockdown (CloudWatch events and Lambda)
E.>	General Best Practice
a.	Look at Inspector for findings/remediations
Remediation Actions
Before we get ahead of ourselves, we must stop any further actions from taking place. This requires removing the foothold in our environment, revoking any active credentials or removing those credentials capabilities, and blocking further actions by the attacker. Based upon your existing work, you’ve implemented the first step by using the CloudWatch Event rule to trigger the Lambda function to update the NACL for the instance. Let’s look at what changed.
1. In the AWS Management Console go to Config
2. Click Cancel as Config is already enabled
3. Click Resources in the left navigation
4. Select Tag
5. Enter the following Key Pair
a.	Name: threat-detection-wksp
6. Click on the Config timeline for the EC2 NetworkAcl
7. Click on Change
8. Evaluate the change to see the updated NACL
Now we’ve stopped the active session from the attacker. Next, we will stop the attacker, or anyone else, from coming from a different IP. 
9. In the AWS Management Console go to EC2 Console
10. Find the running instances and find the active, compromised instance
a.	You can do this by using the instance id you wrote down from the GuardDuty findings earlier
11. On the bottom, find and click on the Security Group for the compromised instance
12. Click on the Security Group and review the inbound rules
a.	Notice the large number of IP’s that can log in
13. Modify the inbound rule to only allow access from your current IP Address (Aa)
a.	How can you set the IP address without explicitly knowing your IP?
Now that the attacker can’t SSH into the compromised machine, we need to rotate the keys and disable their permissions. Before we do that though, let’s revoke all accesses held by the previous credentials.
14. Browse to the IAM console
15. Click Roles and find the role threat-detection-wksp-compromised-ec2 (this is the role attached to the compromised instance)
16. Click on the Revoke sessions tab
17. Click on Revoke active sessions, click the check box and then click Revoke active sessions. (Ca)
Now that those credentials won’t work we need to rotate the existing server credentials. In order to change the IAM credentials on the server, we must Stop and Start the server. A simple reboot will not change the keys.
18. Go back to the EC2 Console
19. To see the keys currently active on the instance, click on Run Command on the left hand navigation
20. Click “Run a command”
21. Select “AWS-RunShellScript”. The instance in your account should already be selected
22. In “Commands” type: curl http://169.254.169.254/latest/meta-data/iam/security-credentials/threat-detection-wksp-compromised-ec2
23. Click Run
24. Back at the console click Output on the bottom of the screen once the Status is “Success”.
25. Click View Output
26. Make note of the “AccessKeyId” and “SecretAccessKey”
27. In the EC2 console Stop the Instance.
28. Wait for the Instance State to say “Stopped”
29. Start the instance. (Ca)
30. Repeat steps 18-25
a.	Notice the keys are different.
b.	If you want, try again after rebooting the server. The keys will stay the same.

Clearing the suspected malware is out of scope for now, but we will establish alerts for this later. This is a good use case for auto-scaling groups and golden-image AMI’s, but that is out of scope for this workshop. With the EC2 instance isolated and the IAM credentials revoked, we need to stop external access to the S3 bucket next. Before we restore the previous configuration, we can quickly make the bucket only accessible from inside the VPC. Then we can re-enable encryption.

31. First, check the configuration of the S3 Endpoint in your environment by going to VPC, Endpoints.
a.	Make note of the VPC Endpoint ID
32. Check the Policy on the bottom tab to notice all access is allowed
33. Check the Route Tables to see who is using the Endpoint.
a.	Notice that routing to S3 is through the VPC endpoint
34. Now to update the S3 bucket policy click on S3 from the Services Drop down
35. Click the bucket that ends in “-data”
36. Click on the Permissions Tab
37. Click Bucket Policy
38. Update the Bucket Policy with the following text:
```
{
  "Version": "2012-10-17",
  "Id": "Policy1415115909152",
  "Statement": [
    {
      "Sid": "Access-to-specific-VPCE-only",
      "Principal": "*",
      "Action": "s3:*",
      "Effect": "Deny",
      "Resource": ["arn:aws:s3:::{BUCKETNAME}",
                   "arn:aws:s3::: {BUCKETNAME}/*"],
      "Condition": {
        "StringNotEquals": {
          "aws:sourceVpce": "{VPCENDPOINTNAME}"
        }
      }
    }
  ]
}
```
39. Click Save. Now regardless of the bucket’s Access Control List, if traffic isn’t coming from the VPC Endpoint it will be denied. This also provides more security and better pricing for legitimate traffic. (Db)
40. To quickly restore the previous configuration for this bucket, we start by going to Config under the Services drop down.
41. In the Dashboard, click on S3 Bucket
42. Click the bucket name
43. Click on “Change” under the previous configuration in the slider
a.	Make note of the changes to Permissions
44. Click on Manage Resource in the top right. This will take you to the S3 console.
45. Click on the Permissions Tab
46. Remove Public Access based on Config finding
47. Click on the Properties Tab
48. Re-enable S3 Default AES-256 encryption based on the Config update and Macie’s earlier alert (Da)
With the configuration reestablished we will focus on alerts and automated remediation should the attacker try again. In Module 1 we put some of this in place. This is where the CloudWatch alerts tied to Lambda functions come into play.
49. Go to the CloudWatch console
50. Click on Event -> Rules
51. Read through the 3 Rules
a.	Notice each Rule creates an alert or executes a Lambda
52. Go to the Lambda service from the Services drop down
53. Review the existing lambda functions
a.	You can examine the Lambda functions at your leisure to see how they handle events

There are additional protections you can put in place. Look at these following actions and see how you would implement them.

•	Update IAM Policy to have restrictive policy
•	Create Config rule to alert to any changes to IAM policy (Cb)
•	Create Lambda function for setting Bucket Encryption
o	Input is the Basic Alert you created
o	Lambda looks up bucket name in CloudTrail
o	Runs command to add Bucket encryption using default AES if encryption is not there
•	Create CloudWatch alert to push to Lambda on Macie finding of Encryption removed (Dc)
o	If I change bucket encryption do I see a S3 object encryption change {CopyObject call}
•	Check for any other issues using Inspector
o	Evaluate findings created by Lambda (Ea)
o	Use Run command to delete hacked User
o	Use Run command to change authentication for user to keypair (Ac)
