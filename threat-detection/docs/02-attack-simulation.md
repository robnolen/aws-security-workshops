# Module 2: Attack Simulation

Now that you have all the detective controls setup and some automated remediations, you'll be running another CloudFormation template which will simulate a variety of findings.

### Agenda

1. Run the 2nd CloudFormation template – 5 min
2. Threat Detection & Remediation Introduction Presentation – 25 min

## Deploy the CloudFormation Template

To initiate the attack simulation you will need to run the module 2 CloudFormation template: 

Region| Launch
------|-----
US West 2 (Oregon) | [![Launch Module 2 in us-west-2](../images/launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=ThreatDetectionWksp-Attack&templateURL=https://s3-us-west-2.amazonaws.com/sa-security-specialist-workshops-us-west-2/02-attack-simulation.yml)

### Launch Instructions

1. Click the **Launch Stack** button above.  This will automatically take you to the console to run the template.  The file for the CloudFormation template (**01-environment-setup.yml**) is also available in the [templates](../templates/) folder if you'd like to download it and manually upload it to create a stack.
2. Leave everything on the next pages as the **Default** settings. 
3. Finally, acknowledge the template will create IAM roles and click **Create**

![IAM Capabilities](../images/iam-capabilities.png)

This will bring you back to the CloudFormation console. You can refresh the page to see the stack starting to create. Before moving on, make sure the stack is in a **CREATE_COMPLETE** status as shown below.

![Stack Complete](../images/01-stack-complete.png)

### Threat Detection and Remediation Presentation

Here is the diagram of the setup after the module 2 CloudFormation stack is created:

![Module 2 Diagram](../images/02-diagram-module2-3.png)