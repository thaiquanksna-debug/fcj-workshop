---
title: "Security, Cost Optimization and Cleanup"
date: 2026-07-04
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

# Security Controls Implemented

The workshop implements multiple layers of security controls to ensure that the internal backend workload is not directly exposed to the public Internet, while still maintaining administration, monitoring, and auditability after deployment.

The main security controls include:

~~~text
No public EC2 IP
No SSH key pair for EC2 administration
No public SSH/RDP ingress
AWS Systems Manager Session Manager for EC2 access
Private RDS PostgreSQL
RDS public access disabled
RDS access restricted by Security Group
RDS storage encrypted with AWS KMS
EC2 root EBS encryption
S3 evidence/log bucket encryption
VPC Endpoints for private AWS service access
SQS Processing Queue and Dead Letter Queue
CloudWatch Alarm integrated with SNS Email
CloudTrail for API activity logging
AWS Config for resource configuration tracking
VPC Flow Logs for network traffic metadata
Terraform for Infrastructure as Code
OPA/Rego policy gate before Terraform apply
~~~

These controls support three main objectives:

1. **Reducing public exposure:** The EC2 Worker has no public IP address, public SSH is not used, and RDS is not publicly accessible.
2. **Improving secure operations:** EC2 administration is performed through Session Manager, jobs are processed through SQS, and failed jobs are isolated in the Dead Letter Queue.
3. **Generating audit evidence:** CloudTrail, AWS Config, VPC Flow Logs, CloudWatch, and S3 provide evidence for post-deployment review.

# Cost Controls

The lab version is designed with cost control in mind. It does not over-provision resources beyond what is required to prove the project scope.

The main cost controls include:

~~~text
One EC2 Worker for lab deployment
Single-AZ RDS PostgreSQL for cost control
No NAT Gateway
No public load balancer
VPC Endpoints used only for required AWS services
SQS and EventBridge used at low demo traffic volume
CloudWatch log retention kept short for lab usage
Terraform destroy after evidence collection
~~~

The resources that can easily generate ongoing cost if left running include:

- Amazon EC2;
- Amazon RDS PostgreSQL;
- VPC Interface Endpoints;
- AWS KMS;
- CloudWatch Logs, Metrics, and Alarms;
- AWS Config;
- VPC Flow Logs;
- S3 storage.

Because the project uses hourly billed resources such as EC2, RDS, and VPC Interface Endpoints, cleanup is a mandatory step after evidence collection.

The estimated cost for the lab configuration, if running continuously 24/7 in `us-east-1`, is approximately **85–95 USD/month**. During the actual demo, the cost can be much lower because the infrastructure is deployed temporarily, validated, documented with screenshots, and then destroyed.

# Cleanup

After collecting enough evidence for the report, the infrastructure should be destroyed using Terraform to avoid unnecessary AWS charges.

Run the following commands in Windows PowerShell:

~~~powershell
cd D:\mvp_private_by_default_architecture
aws sso login --profile mvp --no-browser
cd D:\mvp_private_by_default_architecture\infra\envs\mvp
terraform plan -destroy -out destroy.binary
terraform apply destroy.binary
~~~

When Terraform asks for confirmation, type:

~~~text
yes
~~~

The expected result is:

~~~text
Destroy complete!
~~~

# Post-Cleanup Verification

After Terraform destroy is completed, the AWS Console should be checked to confirm that the main resources have been removed.

The resources to verify include:

- EC2 Worker;
- RDS PostgreSQL database;
- SQS Processing Queue;
- SQS Dead Letter Queue;
- EventBridge schedule;
- SNS topic and email subscription;
- CloudWatch Alarms;
- VPC Endpoints;
- S3 evidence/log bucket;
- CloudTrail;
- AWS Config;
- VPC Flow Logs.

Some resources such as KMS keys may not be deleted immediately. They may enter a scheduled deletion state according to the configured deletion window. This is normal AWS KMS behavior.

If AWS Billing still shows unexpected cost after cleanup, the common resources to check are RDS snapshots, NAT Gateway, VPC Endpoints, CloudWatch Logs, AWS Config Recorder, S3 buckets, and KMS keys.

# Workshop Chapter Conclusion

The Workshop chapter has presented the full deployment lifecycle of **Secure Internal Job Processing Platform on AWS**.

The implementation process includes:

1. preparing the local environment and AWS access;
2. building the infrastructure using Terraform;
3. validating the Terraform plan using OPA/Rego;
4. deploying the workload in a private-first AWS environment;
5. validating the job-processing flow through EventBridge, Amazon SQS, and the EC2 Worker;
6. monitoring the system using CloudWatch and SNS;
7. collecting security, operational, and audit evidence;
8. cleaning up AWS resources to control cost.

The deployment results prove that the project does not stop at an architecture diagram. The system was actually deployed on AWS, with a worker and database that are not publicly exposed to the Internet, a queue-based asynchronous job-processing flow, a Dead Letter Queue for failed-job isolation, CloudWatch/SNS for alerting, CloudTrail/AWS Config/VPC Flow Logs for audit support, and Terraform/OPA for infrastructure control through code.

In other words, the workshop demonstrates that an internal backend workload on AWS can be deployed in a secure, verifiable, and cost-controlled manner, with a clear cleanup process after the demo is completed.