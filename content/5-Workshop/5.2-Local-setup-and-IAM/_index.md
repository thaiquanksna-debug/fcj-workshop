---
title: "Local setup and AWS IAM access"
date: 2026-07-04
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Goal

This section prepares a Windows machine, VS Code, AWS CLI SSO profile, and the AWS permission set required to deploy the workshop. The reader starts with no local source code.

{{% notice warning %}}
Run Windows commands in **Windows PowerShell**. Later, when the workshop opens EC2 through Session Manager, commands marked as **EC2 Linux terminal** must be run inside that browser terminal. Do not mix PowerShell backticks with Linux shell backslashes.
{{% /notice %}}

# 1. Install local tools

Open **Windows PowerShell as Administrator** and install the required tools:

~~~powershell
winget install --id Microsoft.VisualStudioCode -e
winget install --id Git.Git -e
winget install --id Amazon.AWSCLI -e
winget install --id Hashicorp.Terraform -e
~~~


Install OPA with a fixed Windows binary:

~~~powershell
New-Item -ItemType Directory -Force C:\Tools\opa | Out-Null
Invoke-WebRequest -Uri "https://github.com/open-policy-agent/opa/releases/download/v1.7.1/opa_windows_amd64.exe" -OutFile "C:\Tools\opa\opa.exe"
$CurrentUserPath = [Environment]::GetEnvironmentVariable("Path", "User")
if ($CurrentUserPath -notlike "*C:\Tools\opa*") {{
  [Environment]::SetEnvironmentVariable("Path", "$CurrentUserPath;C:\Tools\opa", "User")
}}
Write-Host "Close this PowerShell window and open a new PowerShell window before checking opa version."
~~~


Close PowerShell, open a new PowerShell window, then verify:

~~~powershell
git --version
aws --version
terraform version
opa version
code --version
~~~


Expected result: every command prints a version. If one command is not found, install that tool again before continuing.

# 2. Create the AWS IAM Identity Center user

Open AWS Console with an administrator account.

1. Search for **IAM Identity Center**.
2. Open **IAM Identity Center**.
![Photo](/images/5-Workshop/private-by-default/screen21.png)

3. If IAM Identity Center is not enabled, click **Enable**.
![Photo](/images/5-Workshop/private-by-default/screen2,2.jpg)
![Photo](/images/5-Workshop/private-by-default/screen2,3.jpg)

4. Go to **Users**.
5. Click **Add user**.
![Photo](/images/5-Workshop/private-by-default/screen2,4.jpg)
6. User name: `mvp-builder`.
7. Email: use the email that will sign in to AWS.
8. Complete the user creation.
![Photo](/images/5-Workshop/private-by-default/screen2,5.jpg)

# 3. Create the permission set

1. In **IAM Identity Center**, go to **Permission sets**.
![Photo](/images/5-Workshop/private-by-default/screen2,6.png)

2. Click **Create permission set**.
![Photo](/images/5-Workshop/private-by-default/screen2,7.png)

3. Choose **Custom permission set**.
![Photo](/images/5-Workshop/private-by-default/screen2,8.jpg)
4. Permission set name: `PrivateWorkloadTerraformOperator`.
5. Session duration: choose 4 hours or 8 hours.
6. Create the permission set.
![Photo](/images/5-Workshop/private-by-default/screen2,9.png)
7. Open `PrivateWorkloadTerraformOperator`.
8. Go to **Permissions Set and click MVPTerraformOperator**.
9. Click **Add inline policy** or **Edit inline policy**.
![Photo](/images/5-Workshop/private-by-default/screen2,10.jpg)

10. Choose the **JSON** tab.
11. Delete any existing placeholder JSON.
12. Paste the full policy below.
Full inline policy:

~~~json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "PrivateWorkloadReadDiscovery",
			"Effect": "Allow",
			"Action": [
				"sts:GetCallerIdentity",
				"ec2:Describe*",
				"rds:Describe*",
				"kms:List*",
				"kms:Describe*",
				"kms:GetKeyPolicy",
				"kms:GetKeyRotationStatus",
				"logs:Describe*",
				"logs:ListTagsForResource",
				"cloudtrail:Describe*",
				"cloudtrail:Get*",
				"cloudtrail:ListTags",
				"config:Describe*",
				"config:Get*",
				"cloudwatch:Describe*",
				"cloudwatch:List*",
				"sqs:ListQueues",
				"sqs:GetQueueAttributes",
				"sqs:GetQueueUrl",
				"sqs:ListQueueTags",
				"sns:ListTopics",
				"sns:ListSubscriptions",
				"sns:ListSubscriptionsByTopic",
				"sns:GetTopicAttributes",
				"sns:GetSubscriptionAttributes",
				"sns:ListTagsForResource",
				"scheduler:ListSchedules",
				"scheduler:GetSchedule",
				"scheduler:ListScheduleGroups",
				"scheduler:ListTagsForResource",
				"events:DescribeRule",
				"events:ListRules",
				"events:ListTargetsByRule",
				"events:ListTagsForResource",
				"s3:ListAllMyBuckets",
				"s3:GetBucketLocation",
				"s3:GetBucketPolicy",
				"s3:GetBucketVersioning",
				"s3:GetBucketPublicAccessBlock",
				"s3:GetEncryptionConfiguration",
				"s3:GetBucketTagging",
				"s3:GetBucketCORS",
				"s3:GetBucketWebsite",
				"s3:GetBucketLogging",
				"s3:GetBucketNotification",
				"s3:GetBucketRequestPayment",
				"s3:GetLifecycleConfiguration",
				"s3:GetReplicationConfiguration",
				"s3:GetAccelerateConfiguration",
				"s3:GetBucketObjectLockConfiguration",
				"s3:GetBucketOwnershipControls",
				"iam:Get*",
				"iam:List*",
				"ssm:Describe*",
				"ssm:GetParameter",
				"ssm:GetParameters",
				"ssm:GetParametersByPath",
				"ssm:ListTagsForResource"
			],
			"Resource": "*"
		},
		{
			"Sid": "PrivateWorkloadNetwork",
			"Effect": "Allow",
			"Action": [
				"ec2:CreateVpc",
				"ec2:DeleteVpc",
				"ec2:ModifyVpcAttribute",
				"ec2:CreateSubnet",
				"ec2:DeleteSubnet",
				"ec2:ModifySubnetAttribute",
				"ec2:CreateRouteTable",
				"ec2:DeleteRouteTable",
				"ec2:AssociateRouteTable",
				"ec2:DisassociateRouteTable",
				"ec2:ReplaceRouteTableAssociation",
				"ec2:CreateRoute",
				"ec2:DeleteRoute",
				"ec2:CreateSecurityGroup",
				"ec2:DeleteSecurityGroup",
				"ec2:AuthorizeSecurityGroupIngress",
				"ec2:RevokeSecurityGroupIngress",
				"ec2:AuthorizeSecurityGroupEgress",
				"ec2:RevokeSecurityGroupEgress",
				"ec2:CreateVpcEndpoint",
				"ec2:DeleteVpcEndpoints",
				"ec2:ModifyVpcEndpoint",
				"ec2:CreateFlowLogs",
				"ec2:DeleteFlowLogs",
				"ec2:CreateTags",
				"ec2:DeleteTags"
			],
			"Resource": "*"
		},
		{
			"Sid": "PrivateWorkloadEC2PrivateWorker",
			"Effect": "Allow",
			"Action": [
				"ec2:RunInstances",
				"ec2:TerminateInstances",
				"ec2:StopInstances",
				"ec2:StartInstances",
				"ec2:RebootInstances",
				"ec2:ModifyInstanceAttribute",
				"ec2:AssociateIamInstanceProfile",
				"ec2:ReplaceIamInstanceProfileAssociation",
				"ec2:DisassociateIamInstanceProfile"
			],
			"Resource": "*"
		},
		{
			"Sid": "PrivateWorkloadRDSPostgreSQL",
			"Effect": "Allow",
			"Action": [
				"rds:CreateDBSubnetGroup",
				"rds:DeleteDBSubnetGroup",
				"rds:ModifyDBSubnetGroup",
				"rds:CreateDBInstance",
				"rds:DeleteDBInstance",
				"rds:ModifyDBInstance",
				"rds:AddTagsToResource",
				"rds:RemoveTagsFromResource",
				"rds:ListTagsForResource"
			],
			"Resource": "*"
		},
		{
			"Sid": "PrivateWorkloadKMS",
			"Effect": "Allow",
			"Action": [
				"kms:CreateKey",
				"kms:ScheduleKeyDeletion",
				"kms:CancelKeyDeletion",
				"kms:EnableKeyRotation",
				"kms:DisableKeyRotation",
				"kms:GetKeyRotationStatus",
				"kms:CreateAlias",
				"kms:DeleteAlias",
				"kms:UpdateAlias",
				"kms:UpdateKeyDescription",
				"kms:PutKeyPolicy",
				"kms:GetKeyPolicy",
				"kms:TagResource",
				"kms:UntagResource",
				"kms:ListResourceTags",
				"kms:CreateGrant",
				"kms:ListGrants",
				"kms:RevokeGrant",
				"kms:Encrypt",
				"kms:Decrypt",
				"kms:ReEncryptFrom",
				"kms:ReEncryptTo",
				"kms:GenerateDataKey",
				"kms:GenerateDataKeyWithoutPlaintext",
				"kms:GenerateDataKeyPair",
				"kms:GenerateDataKeyPairWithoutPlaintext"
			],
			"Resource": "*"
		},
		{
			"Sid": "PrivateWorkloadSSMParameterStore",
			"Effect": "Allow",
			"Action": [
				"ssm:PutParameter",
				"ssm:DeleteParameter",
				"ssm:AddTagsToResource",
				"ssm:RemoveTagsFromResource",
				"ssm:ListTagsForResource"
			],
			"Resource": "arn:aws:ssm:us-east-1:*:parameter/mvp/*"
		},
		{
			"Sid": "PrivateWorkloadLoggingAudit",
			"Effect": "Allow",
			"Action": [
				"cloudtrail:CreateTrail",
				"cloudtrail:UpdateTrail",
				"cloudtrail:DeleteTrail",
				"cloudtrail:StartLogging",
				"cloudtrail:StopLogging",
				"cloudtrail:PutEventSelectors",
				"cloudtrail:AddTags",
				"cloudtrail:RemoveTags",
				"cloudtrail:ListTags",
				"logs:CreateLogGroup",
				"logs:DeleteLogGroup",
				"logs:PutRetentionPolicy",
				"logs:CreateLogStream",
				"logs:PutLogEvents",
				"logs:TagResource",
				"logs:UntagResource",
				"logs:AssociateKmsKey",
				"logs:DisassociateKmsKey",
				"config:PutConfigurationRecorder",
				"config:DeleteConfigurationRecorder",
				"config:StartConfigurationRecorder",
				"config:StopConfigurationRecorder",
				"config:PutDeliveryChannel",
				"config:DeleteDeliveryChannel"
			],
			"Resource": "*"
		},
		{
			"Sid": "PrivateWorkloadCloudWatchAlarms",
			"Effect": "Allow",
			"Action": [
				"cloudwatch:PutMetricAlarm",
				"cloudwatch:DeleteAlarms",
				"cloudwatch:EnableAlarmActions",
				"cloudwatch:DisableAlarmActions",
				"cloudwatch:SetAlarmState",
				"cloudwatch:TagResource",
				"cloudwatch:UntagResource"
			],
			"Resource": "*"
		},
		{
			"Sid": "PrivateWorkloadS3",
			"Effect": "Allow",
			"Action": [
				"s3:CreateBucket",
				"s3:DeleteBucket",
				"s3:PutEncryptionConfiguration",
				"s3:GetEncryptionConfiguration",
				"s3:PutBucketVersioning",
				"s3:PutBucketPolicy",
				"s3:PutBucketPublicAccessBlock",
				"s3:GetBucketPublicAccessBlock",
				"s3:GetBucketPolicy",
				"s3:GetBucketVersioning",
				"s3:GetBucketLocation",
				"s3:GetBucketTagging",
				"s3:PutBucketTagging",
				"s3:GetBucketAcl",
				"s3:PutBucketAcl",
				"s3:GetBucketCORS",
				"s3:GetBucketWebsite",
				"s3:GetBucketLogging",
				"s3:GetBucketNotification",
				"s3:GetBucketRequestPayment",
				"s3:GetLifecycleConfiguration",
				"s3:GetReplicationConfiguration",
				"s3:GetAccelerateConfiguration",
				"s3:GetBucketObjectLockConfiguration",
				"s3:GetBucketOwnershipControls",
				"s3:ListBucket",
				"s3:GetObject",
				"s3:PutObject",
				"s3:DeleteObject"
			],
			"Resource": [
				"arn:aws:s3:::private-by-default-mvp-*",
				"arn:aws:s3:::private-by-default-mvp-*/*"
			]
		},
		{
			"Sid": "PrivateWorkloadSQSBusinessFlow",
			"Effect": "Allow",
			"Action": [
				"sqs:CreateQueue",
				"sqs:DeleteQueue",
				"sqs:GetQueueAttributes",
				"sqs:SetQueueAttributes",
				"sqs:GetQueueUrl",
				"sqs:ListQueues",
				"sqs:ListQueueTags",
				"sqs:TagQueue",
				"sqs:UntagQueue",
				"sqs:SendMessage",
				"sqs:ReceiveMessage",
				"sqs:DeleteMessage",
				"sqs:ChangeMessageVisibility",
				"sqs:PurgeQueue"
			],
			"Resource": [
				"arn:aws:sqs:us-east-1:*:private-by-default-mvp-*"
			]
		},
		{
			"Sid": "PrivateWorkloadSNSAlerts",
			"Effect": "Allow",
			"Action": [
				"sns:CreateTopic",
				"sns:DeleteTopic",
				"sns:GetTopicAttributes",
				"sns:SetTopicAttributes",
				"sns:ListTopics",
				"sns:ListSubscriptions",
				"sns:ListSubscriptionsByTopic",
				"sns:ListTagsForResource",
				"sns:TagResource",
				"sns:UntagResource",
				"sns:Subscribe",
				"sns:Unsubscribe",
				"sns:GetSubscriptionAttributes",
				"sns:SetSubscriptionAttributes",
				"sns:Publish"
			],
			"Resource": [
				"arn:aws:sns:us-east-1:*:private-by-default-mvp-*"
			]
		},
		{
			"Sid": "PrivateWorkloadSNSReadGlobal",
			"Effect": "Allow",
			"Action": [
				"sns:ListTopics",
				"sns:ListSubscriptions"
			],
			"Resource": "*"
		},
		{
			"Sid": "PrivateWorkloadSchedulerBusinessFlow",
			"Effect": "Allow",
			"Action": [
				"scheduler:CreateSchedule",
				"scheduler:DeleteSchedule",
				"scheduler:GetSchedule",
				"scheduler:UpdateSchedule",
				"scheduler:ListSchedules",
				"scheduler:ListScheduleGroups",
				"scheduler:TagResource",
				"scheduler:UntagResource",
				"scheduler:ListTagsForResource"
			],
			"Resource": [
				"arn:aws:scheduler:us-east-1:*:schedule/*/private-by-default-mvp-*",
				"arn:aws:scheduler:us-east-1:*:schedule-group/*"
			]
		},
		{
			"Sid": "PrivateWorkloadEventBridgeRuleAlternative",
			"Effect": "Allow",
			"Action": [
				"events:PutRule",
				"events:DeleteRule",
				"events:DescribeRule",
				"events:EnableRule",
				"events:DisableRule",
				"events:PutTargets",
				"events:RemoveTargets",
				"events:ListRules",
				"events:ListTargetsByRule",
				"events:TagResource",
				"events:UntagResource",
				"events:ListTagsForResource"
			],
			"Resource": [
				"arn:aws:events:us-east-1:*:rule/private-by-default-mvp-*"
			]
		},
		{
			"Sid": "PrivateWorkloadIAMServiceRoles",
			"Effect": "Allow",
			"Action": [
				"iam:CreateRole",
				"iam:DeleteRole",
				"iam:UpdateAssumeRolePolicy",
				"iam:AttachRolePolicy",
				"iam:DetachRolePolicy",
				"iam:PutRolePolicy",
				"iam:DeleteRolePolicy",
				"iam:TagRole",
				"iam:UntagRole"
			],
			"Resource": [
				"arn:aws:iam::*:role/private-by-default-mvp-*",
				"arn:aws:iam::*:role/aws-service-role/*"
			]
		},
		{
			"Sid": "PrivateWorkloadIAMInstanceProfilesForPrivateWorker",
			"Effect": "Allow",
			"Action": [
				"iam:CreateInstanceProfile",
				"iam:GetInstanceProfile",
				"iam:DeleteInstanceProfile",
				"iam:AddRoleToInstanceProfile",
				"iam:RemoveRoleFromInstanceProfile",
				"iam:TagInstanceProfile",
				"iam:UntagInstanceProfile",
				"iam:ListInstanceProfilesForRole"
			],
			"Resource": [
				"arn:aws:iam::*:instance-profile/private-by-default-mvp-*",
				"arn:aws:iam::*:role/private-by-default-mvp-*"
			]
		},
		{
			"Sid": "PrivateWorkloadPassRoleToAWSServiceOnly",
			"Effect": "Allow",
			"Action": "iam:PassRole",
			"Resource": "arn:aws:iam::*:role/private-by-default-mvp-*",
			"Condition": {
				"StringEquals": {
					"iam:PassedToService": [
						"ec2.amazonaws.com",
						"rds.amazonaws.com",
						"config.amazonaws.com",
						"vpc-flow-logs.amazonaws.com",
						"scheduler.amazonaws.com",
						"events.amazonaws.com"
					]
				}
			}
		}
	]
}
~~~

13. Create or Save changes.
![Photo](/images/5-Workshop/private-by-default/screen2,11.jpg)



# 4. Assign the user to the AWS account

1. In **IAM Identity Center**, go to **AWS accounts**.
2. Select the target AWS account.
3. Click **Assign users or groups**.
![Photo](/images/5-Workshop/private-by-default/screen4,1.png)

4. Select user `mvp-builder`.
![Photo](images/5-Workshop/private-by-default/screen4,2.png)
5. Select permission set `PrivateWorkloadTerraformOperator`.
![Photo](/images/5-Workshop/private-by-default/screen4,3.png)
6. Submit the assignment.
![Photo](/images/5-Workshop/private-by-default/screen4,4.png)

# 5. Configure the AWS CLI SSO profile

In IAM Identity Center, open **Settings** and copy the **AWS access portal URL**.

Run this in Windows PowerShell:

~~~powershell
aws configure sso
~~~


Use these answers:

~~~text
SSO session name: mvp
SSO start URL: paste the AWS access portal URL from IAM Identity Center
SSO region: us-east-1
SSO registration scopes: press Enter
Account: select your AWS account
Role: PrivateWorkloadTerraformOperator
CLI default client Region: us-east-1
CLI default output format: json
CLI profile name: mvp
~~~


Then test the profile:

~~~powershell
aws sso logout
aws sso login --profile mvp --no-browser
aws sts get-caller-identity --profile mvp
~~~


Expected result: the command prints your AWS account ID. Continue only after this works.
6. Verify local environment before continuing

Before downloading the source code, verify that all required tools are working correctly.

Run:

git --version
aws --version
terraform version
opa version
code --version

Expected result:

git version x.x.x
aws-cli/x.x.x
Terraform v1.x.x
Version: x.x.x
1.xx.x

If any command returns:

command not found

or

is not recognized as an internal or external command

close PowerShell, reopen it, and verify that the tool was installed correctly.

7. Recommended AWS region

This workshop was tested in:

us-east-1 (N. Virginia)

Using a different region may require modifications to:

AMI IDs
Service availability
Pricing estimates
VPC Endpoint service names

For reproducibility, use:

us-east-1

throughout the workshop.

8. Estimated deployment cost

This project is intentionally designed as a low-cost secure infrastructure demonstration.

Main cost contributors:

Service	Estimated Monthly Cost
EC2 Private Worker	8–12 USD
RDS PostgreSQL	12–20 USD
VPC Endpoints	45–60 USD
CloudWatch / SNS / SQS	1–3 USD
CloudTrail / Config / Logs	2–5 USD

Estimated total:

85–95 USD/month

when running continuously 24/7.

Actual workshop usage is usually much lower because the environment only runs for a limited time.

Many students observe only a few AWS credits consumed because:

deployment exists for a short period
free tier credits may apply
resources are deleted after testing
9. What success looks like

At the end of the workshop, the reader should be able to verify:

Security

✓ Worker has no public IP

✓ Database has no public endpoint

✓ SSH is not used

✓ Session Manager access works

✓ Traffic uses VPC Endpoints

Infrastructure

✓ Terraform deploy succeeds

✓ Terraform destroy succeeds

✓ OPA policy validation succeeds

Business Flow

✓ Message arrives in SQS

✓ Worker receives the message

✓ Worker processes the job

✓ CloudWatch receives logs

✓ Failed message reaches DLQ

✓ SNS email notification works

Audit

✓ CloudTrail records activity

✓ AWS Config records resources

✓ VPC Flow Logs exist

✓ Evidence screenshots collected

10. Before moving to the next section

Confirm:

[ ] VS Code installed

[ ] Git installed

[ ] AWS CLI installed

[ ] Terraform installed

[ ] OPA installed

[ ] IAM Identity Center user created

[ ] Permission set assigned

[ ] AWS SSO login successful

[ ] aws sts get-caller-identity works

Only continue to Section 5.3 after every item above is complete.