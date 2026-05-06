
In this hands-on project, we'll build a real-time AI-powered threat detection and response system on AWS using GuardDuty, SNS, and Lambda. This setup will enable you to automatically detect threats and take immediate action, significantly improving your security posture. Simulate abnormal behavior in an AWS environment and use AI-powered tools to detect, respond, and notify you in real time.

🛠 Tech Stack:
This project showcases a real-time AI-powered security pipeline using:

Amazon CloudTrail (log API activity)
Amazon GuardDuty (AI-based threat detection)
Amazon EventBridge (trigger on GuardDuty findings)
AWS Lambda (automated response)
Amazon SNS (send real-time email/SMS alerts)
Simulated GuardDuty findings, trigger SNS alerts and a Lambda function that sends a clean, human-readable security alert.

🔧 Prerequisites
✅ An AWS account
✅ AWS CLI configured

➡️ Step 1 - Enable CloudTrail
Amazon CloudTrail records all AWS API calls and activity in your account. GuardDuty analyzes these logs.

How to do it:

Go to the CloudTrail console
If it’s not already enabled: Click “Create trail”
Choose “Management events”
Choose to log to an S3 bucket
Click “Create trail”
✅ Now your account logs all actions taken by users, roles, and services.

➡️ Step 2 - Enable Amazon GuardDuty
Amazon GuardDuty will analyze CloudTrail, DNS, VPC Flow Logs, and more using ML + threat intel to detect suspicious behavior.

How to do it:

Go to the GuardDuty console
Click “Enable GuardDuty”
Wait 5-10 mins, it starts analyzing logs.
✅ GuardDuty is now scanning your account for threats like credential theft, unusual login behavior, port scanning, and more.

➡️ Step 3 - Set Up an SNS Topic for Notifications
Next, we'll create an SNS topic to send alerts to you or your security team whenever a threat is detected.

How to do it:

Go to Amazon SNS → Topics → Create topic
Type: Standard
Name: GuardDuty-Threat-Alerts
Click Create topic
Click “Create subscription”
Protocol: Email
Endpoint: your email address
Confirm the email by coying the link in your inbox:
Copy the Confirm subscription URL
Go back to your subscription page, click on Confirm subscription button
Paste the URL in the dialog box, and click Confirm subscription.
✅ SNS is now ready to send you alerts.

➡️ Step 4 - Create a Lambda Function - Our Alert Processor
We'll create a Lambda function that takes the complicated JSON output from GuardDuty and turns it into a simple message.

🔹 Create the Lambda IAM Role:

Go to the IAM console and create a new role:

For the trusted entity, select AWS service, and for the use case, choose Lambda.
On the permissions screen, add the AWSLambdaBasicExecutionRole policy. This allows our function to write logs to CloudWatch, which is essential for debugging.
Name the role something like GuardDuty-Lambda-Role and create it.
🔹 Create the Lambda Function:

Go to the Lambda console and click Create function.
Select Author from scratch.
Function name: GuardDuty-Automated-Response
Runtime: Python 3.13
Architecture: x86_64
Permissions: Choose Use an existing role and select the IAM role you just created.
Click Create function.
Now, let's paste in our Python code. This code will parse the GuardDuty finding, pull out the most important details, and format a clean message.

GuardDuty-Automated-Response.py
🔹 Configure Environment Variables:

In your Lambda function's configuration, go to the Environment variables tab and click Edit.
Add a new variable:
• Key: SNS_TOPIC_ARN
• Value: Paste the ARN of the SNS topic you created in Step 3.
🔹 Attach a Policy to Allow SNS Publish:

In your Lambda function in the AWS Console
Go to Configuration > Permissions
Click the role name GuardDuty-Lambda-Role this will take you to the IAM Role details.
From the IAM Role page, "Add permissions" > "Attach policies"
Choose “Create inline policy” (for full control)
Create and Attach Inline Policy, use the following JSON in the JSON tab:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sns:Publish",
      "Resource": "arn:aws:sns:your-region:your-account-id:your-topic-name"
    }
  ]
}
⚠️ Note: Replace your-region your-account-id your-topic-name with your actual values.

Click Next, give it a name like AllowSNSPublish
Click Create policy
Now this role can successfully publish to your SNS topic!

➡️ Step 5 - Integrate Services with Amazon EventBridge
Now, we'll create an EventBridge rule to trigger our Lambda function and send a notification when GuardDuty detects a specific type of threat.

Go to the Amazon EventBridge console.
In the left navigation pane, click Rules, then Create rule.
Give it a name like GuardDuty-EC2-Threat-Rule
Event bus: default
Rule type: Rule with an event pattern
Click Next.
Event source: AWS events or EventBridge partner events
Event pattern:
• Event source: AWS services
• AWS service: GuardDuty
• Event type: GuardDuty Finding
{
  "source": ["aws.guardduty"],
  "detail-type": ["GuardDuty Finding"],
  "detail": {
    "type": ["Trojan:EC2/BlackholeTraffic"]
  }
}
Click Next.

Select a target:
• Target 1:
• Target types: AWS service
• Select a target: Lambda function
• Function: Select the GuardDuty-Automated-Response function.

Click Next and then Create rule.

🏆 Let's Test It!
We will use the AWS CLI to generate a sample GuardDuty finding that simulates a threat from our test user. This is the most direct way to trigger the entire workflow.

Open your terminal or command prompt that has the AWS CLI configured.
Find your GuardDuty Detector ID:
• Navigate to the GuardDuty console.
• Click on Settings in the left sidebar.
• Copy the Detector ID.
Run the command: Replace YOUR_DETECTOR_ID with your actual Detector ID .
aws guardduty create-sample-findings \
--detector-id YOUR_DETECTOR_ID \
--finding-types "Trojan:EC2/BlackholeTraffic"
This command tells GuardDuty to create a sample finding that mimics anomalous behavior from an EC2 instance that is making outbound connections to known malware, which will trigger our EventBridge rule.

Verification - Check the Results ✅
Now, let's verify that each component of our project worked as expected.

Check for the SNS Notification
• Go to your email inbox that you subscribed to the SNS topic.
📬 You should instantly receive an alert email that looks like this:
🚨 GuardDuty Alert: Threat Detected

🔍 Type: Trojan:EC2/BlackholeTraffic
💡 Description: The EC2 instance is communicating with a blackholed IP...

🖥 Instance ID: i-1234567890abcdef
🌐 Public IP: 198.51.100.10
📍 Region: us-east-1

🧠 Recommendation:
Isolate or stop the EC2 instance and investigate for malware or...
Check GuardDuty Findings
• Go to GuardDuty Console, you'll now see a full list of GuardDuty findings, each row representing a detection event:
• Finding Type:
Recon:EC2/Portscan
Trojan:EC2/BlackholeTraffic
UnauthorizedAccess:EC2/TorClient
• Severity Type: Medium

Check the Lambda Function Logs
• Navigate to the Lambda console and select your GuardDuty-Automated-Response function.
• Click on the Monitor tab, and then View CloudWatch logs.
• Click on the Monitor tab, and then View CloudWatch logs.

