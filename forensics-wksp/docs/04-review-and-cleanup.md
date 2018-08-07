# Module 4: Review and cleanup

In the last module we will have a short discussion of the lab (and discuss exactly what occurred.) We will also go over a number of questions and then provide instructions on how to clean up the lab environment (to prevent future charges in your AWS account.) 

### Agenda

1. Review and discussion – 10 min
2. Cleanup – 5 min

## Architecture Overview

Diagram of the overall lab setup:
![Part 1 Diagram](../images/04-diagram-module4.png)

## What is Really Going On?

In **Module 1** of the lab you setup the initial components of your infrastructure including GuardDuty and a simple automated forensics workflow. Some of the steps required manual configuration, but you also ran a CloudFormation template that set up some of the components.

In **Module 2** you launched a second CloudFormation template that initiated the attack simulated by this lab. The CloudFormation template created two EC2 instances. One instance , named **Malicious Host**, had an EIP attached to it that was added to your GuardDuty custom threat list. Although the **Malicious Host** is in the same VPC as the other instance, for the sake of the scenario (and to prevent the need to submit a penetration testing request) we acted as if it is on the Internet and represented the attacker's computer. The other instance, named **Compromised Instance**, was your web server that was taken over by the **Malicious Host**.

In **Module 3** you performed forensics to investigate the attack by looking at GuardDuty findings, an Inspector assessment, CloudWatch logs, and Athena query results. You also learned about how to remediate the damage and set up some automated remediation for future attacks.  

### What occurred during the attack:	

1. There were two instances created by the Module 2 CloudFormation template. They were in the same VPC but different subnets. The **Malicious Host** represented the attacker that we pretended was on the Internet. The Elastic IP on the **Malicious Host** was in a custom threat list in GuardDuty. The other instance, named **Compromised Instance**, represented the web server that was lifted and shifted into AWS.

2. Although company policy was that only key-based authentication should be enabled for SSH, at some point password authentication for SSH was enabled on the **Compromised Instance**.  
	
	> This misconfiguration is identified in the Inspector scan that is triggered from the GuardDuty finding.

3. The **Malicious Host** performed a brute force SSH password attack against the **Compromised Instance**. The brute force attack had been purposefully designed to be successful.
	
	> **GuardDuty Finding**: UnauthorizedAccess:EC2/SSHBruteForce

4. The SSH brute force attack was successful and the attacker was able to log in to the **Compromised Instance**.
	
	> Successful login is confirmed in CloudWatch Logs (/forensics-wksp/var/log/secure).

5.  The Compromised Instance also had a cron job that continuously pinged the Malicious Host to generate a GuardDuty finding based off the custom threat list.
	
	> **GuardDuty Finding**: UnauthorizedAccess:EC2/MaliciousIPCaller.Custom

6. The API Calls that generated the API findings came from the **Malicious Host**. The calls used the temp creds from the IAM role for EC2 running on the **Malicious Host**. The GuardDuty findings were generated because the EIP attached to the **Malicious Host** was in a custom threat list. 
	
	> **GuardDuty Finding**: Recon:IAMUser/MaliciousIPCaller.Custom
	
	> **GuardDuty Finding**: UnauthorizedAccess:IAMUser/MaliciousIPCaller.Custom

7. A number of CloudWatch Events Rules were evoked by the GuardDuty findings and then these triggered various services:
	1.	**CloudWatch Event Rule**: The generic GuardDuty finding invoked a rule that triggered SNS to send an email.
	2.	**CloudWatch Event Rule**: The SSH brute force attack finding invoked a rule that triggered a Lambda function to (a) block the IP address of the attacker via a NACL change, (b) used a Lambda function to run an Inspector scan on the EC2 instance.
	3. **CloudWatch Event Rule**: The Unauthorized Access Custom MaliciousIP finding invoked a rule that triggered a Lambda function to block the IP address of the attacker via a NACL.

## Clean-up

In order to prevent charges to your account from the resources created during this workshop, we recommend cleaning up the infrastructure that was created. If you plan to keep things running so you can examine the lab a bit more, please remember to do the clean-up when you are done.

### Automated clean-up

To delete the CloudFormation stack, a Bash script, `cleanup.sh`, has been provided. Run as follows:

```
chmod +x cleanup.sh
./cleanup.sh
```

- *Q: I'm using a different AWS CLI profile than the default.*
- A: The script supports a flag to specify a CLI profile that is configured in your `~/.aws/config` file. Do `./teardown.sh -p PROFILE_NAME`.

### Manual clean-up instructions

> You will need to manually delete some resources before you delete the CloudFormation stacks so please do the following steps in order.

1.	Delete the Inspector objects created for the workshop.
	* Go to the [Amazon Inspector](https://us-west-2.console.aws.amazon.com/inspector) console.
	* Click on **Assessment targets** in the navigation pane on the left.
	* Delete all that start with **forensics-wksp**.

2.	Delete the IAM Role for the compromised EC2 instance and the Service-Linked Role for Inspector (if you didn't already have this Role created).
	* Go to [AWS IAM](https://console.aws.amazon.com/iam/) console.
	* Click on **Roles**
	* Search for the role named **forensics-wksp-compromised-ec2**.
	* Click the check box next to it and click **Delete**.
	* Repeat the steps above for the role named **AWSServiceRoleForAmazonInspector**.

3.	Delete all three S3 buckets created by the Module 1 CloudFormation template (the buckets that start with **forensics-wksp** and end with **-data**, **-threatlist** and **-logs**)
	* Go to [Amazon S3](https://s3.console.aws.amazon.com/s3/home?region=us-west-2) console.
	* Click on the appropiate bucket.
	* Click **Delete Bucket**.
	* Copy and paste the name of the bucket (this is an extra verification that you actually want to delete the bucket).
	* Repeat the steps above for all three buckets.

4.	Delete Module 1 and 2 CloudFormation stacks (**BlackHat2018-AwsForensicsWksp-Core** and **BlackHat2018-AwsForensicsWksp-AttackSim**).
	* Go to the [AWS CloudFormation](https://us-west-2.console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks?filter=active) console.
	* Select the appropiate stack.
	* Select **Action**.
	* Click **Delete Stack**.
	* Repeat the steps above for each stack.

	> You do not need to wait for the first stack to delete before you delete the second one.

5.	Delete the GuardDuty custom threat list and disable GuardDuty (if you didn't already have it configured before the workshop)
	* Go to the [Amazon GuardDuty](https://us-west-2.console.aws.amazon.com/guardduty/) console.
	* Click on **Lists** on the left navigation.
	* Click the **X** next to the threat list that starts with **Custom-Threat-List**.
	* Click **Settings** in the navigation pane on the left navigation.
	* Click the check box next to **Disable**.
	* Click **Save settings** and then click **Disable** in the pop-up box.

6.	Delete the manual CloudWatch Event Rule you created and the CloudWatch Logs that were generated.
	* Go to the [AWS CloudWatch](https://us-west-2.console.aws.amazon.com/cloudwatch) console.
	* Click on **Rules** in the navigation pane on the left.
	* Click the radio button next **forensics-wksp-guardduty-finding-maliciousip**.
	* Select **Action** and click **Delete**.
	* Click on **Logs** in the navigation pane on the left.
	* Click the radio button next to **/aws/lambda/forensics-wksp-inspector-role-creation**.
	* Select **Action** and click **Delete log group** and then click **Yes, Delete** in the pop-up box.
	* Repeat for: 
		* **/aws/lambda/forensics-wksp-remediation-inspector**
		* **/aws/lambda/forensics-wksp-remediation-nacl**
		* **/forensics-wksp/var/log/secure** 

7.	Delete the SNS subscription that was created when you subscribed to SNS Topic.
	* Go to the [AWS SNS](https://us-west-2.console.aws.amazon.com/sns) console.
	* Click on **Subscriptions** on the left navigation.
	* Select the check box next to the subscription that shows your e-mail as the Endpoint and has **forensics-wksp** in the **Subscription ARN**.
	* Select **Action** and then click **Delete subscriptions**

## Finished!

Congratulations on completing this workshop! This is the workshop's permanent home, so feel free to revisit as often as you'd like.
