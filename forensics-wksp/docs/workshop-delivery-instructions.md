# Cloud-powered Forensics Workshop
## Workshop Delivery Instructions

Anyone with interest, time and students can deliver this workshop. Everything needed to deliver the content is available in this repository. This workshop is designed take individuals with a 200-300 level understanding of AWS and provide them with the tools and knowledge they need to get started with forensics on AWS. The following directions are meant for the prospective instructor(s).

### Workshop description you can share with students:
Join us for this hands-on workshop where you learn about a number of AWS services involved with threat detection and remediation by walking through some real-world threat scenarios. Learn about the threat detection capabilities of GuardDuty, Macie and Config and the available remediation options. In each hands on scenario we look at methods to investigate and remediate a threat.

This workshop uses the following services:

- Amazon GuardDuty
- Amazon Athena
- Amazon CloudWatch
- AWS Lambda
- AWS Systems Manager
- AWS Step Functions
- AWS CloudTrail

### Initial setup and planning:

1. The workshop was designed to be delivered in just under 2 hours. Even with this timing some students may not be able to finish the entire workshop. When you get to module 4 it is possible some students will still be working through module 3. This is okay. Expect that some students will be more advanced than others, and that some may be entirely new to AWS. The material has been designed in a way that students of all levels can learn and have a good experience.
2. The workshop has been delivered to audiences of between 20 and 150 people and there are no practical limits in audience size.
3. There is about 25 minutes of lecture and presentations and the rest of the time should be devoted to having the students work through the exercises.
4. The students should NOT use production AWS accounts for the labs. Ideally, new accounts should be created before the workshop for each student. Note that newly-created accounts are more likely to get flagged for anomalous behavior, which may result in students not being able to launch EC2 instances. If this happens, they are welcome to create another new account, or otherwise ask if they are okay following along with someone else.
5. The workshop is designed for each student to work individually but there is no reason why the students couldn't work in groups with one person driving.
6. Most importantly, have fun! If something goes wrong, keep going and do the best you can.

### Workshop Directions:

#### Module 1 - Environment build and configuration
	
Time: ~15 minutes
    
1. Start the workshop with the slide presentation up to the Module 1 slide (the one with the repo URL). This is designed to introduce the workshop and get the attendees started on Module 1.
2. The presentation ends with directing the students to begin the Module 1 exercise, which involves running a CloudFormation template and taking a few manual actions.
4. When the students begin the Module 1 exercisem the instructor(s) and facilitators should walk around to assist anyone having difficulty.

#### Module 2 - Attack simulation and detection

Time: ~25 minutes

1. Continue the slide presentation to the Module 2 slide, which comes up fairly soon because it's important to get them creating the CloudFormation stack for the attack simulation early since it takes about 5 minutes to complete and the simulation itself can take up to 20 minutes to run. The template creates a stack that  launches a number of resources in order to simulate an attack.
3. Continue the presentation, stopping at the Module 3 slide. This will be the longest presentation in the workshop and has the side effect of allowing enough time to transpire so that the simulation can complete. Module 2 ends with telling the students to get started with the Module 3 exercises. The exercises represent the bulk of the student activities. 
	
#### Module 3 - Forensics and remediation

Time: ~50 minutes

1. There is no presentation to show for Module 3. At this point the instructor(s) and facilitators should walk around to assist anyone having difficulty with the exercises. 
	
#### Module 4 - Review, discussion, cleanup

Time: ~10 minutes

1. Continue the presentation slides. Module 4 is used to educate the students about how the attack simulation was actually carried out and to do some review questions to test their understanding of the material. A prize or prizes can be given out here for good answers to some of the more difficult questions (like the final question.)
3. Module 4 should end with a reminder to clean up the resources that were created in the workshop. Some students may still be working on Module 3 at this point so just remind them to do the cleanup when they finish. The Module 4 exercises show how to cleanup all the resources.
 