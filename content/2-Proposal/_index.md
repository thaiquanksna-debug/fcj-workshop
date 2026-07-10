---
title: "Proposal"
date: 2026-07-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Secure Internal Job Processing Platform on AWS  
## A secure AWS backend platform for internal batch jobs

### 1. Executive Summary

Secure Internal Job Processing Platform on AWS is a backend system designed for organizations that need to process internal data jobs in a secure cloud environment without exposing workers or databases directly to the Internet.

This project is not a public user-facing website. It is a deployed AWS backend system that simulates internal workloads such as transaction reconciliation, customer data validation, business log processing, audit evidence generation, and scheduled compliance checks.

The architecture uses Amazon VPC for network isolation, Amazon EC2 as the internal worker, Amazon RDS PostgreSQL as the database tier, Amazon SQS for asynchronous job queuing, Amazon EventBridge for scheduled job creation, Amazon CloudWatch and Amazon SNS for monitoring and alerting, Amazon S3 for data/log/evidence storage, and security/governance services such as IAM, KMS, AWS Config, CloudTrail, and Systems Manager.

The system is designed and implemented using a cost-optimized (FinOps) model featuring a single EC2 Worker and an Amazon RDS PostgreSQL Single-AZ instance. This architecture accurately reflects the actual provisioned infrastructure while fully preserving all core strategic objectives: private networking, queue-based processing, proactive monitoring and alerting, audit evidence, and fully automated Infrastructure as Code (IaC).

---

### 2. Problem Statement

#### Current problem

Many organizations have internal data-processing jobs that should not be exposed to the public Internet, such as:

- transaction reconciliation;
- customer CSV validation;
- business log processing;
- audit report generation;
- scheduled compliance checks;
- internal data synchronization or verification.

If these workloads are deployed manually or placed in a loosely controlled public environment, the system may face several risks:

- EC2 or database resources may be exposed to the Internet;
- there may be no queue to preserve jobs when workers fail;
- failed jobs may be difficult to retry or investigate;
- abnormal system behavior may not trigger alerts;
- audit evidence may be missing;
- manual infrastructure changes may cause drift;
- cloud cost may increase due to unnecessary public networking components.

The core problem addressed by this project is: **how to transform cloud security requirements into real AWS infrastructure that can be deployed, validated, and audited.**

#### Proposed solution

The project proposes a private-first internal job-processing platform on AWS. Instead of receiving requests from a public HTTP API, the system receives work through Amazon SQS. Amazon EventBridge creates scheduled jobs and sends messages to SQS. An EC2 Worker inside a private subnet polls messages from SQS and processes the jobs. When data or metadata persistence is required, the worker connects to Amazon RDS PostgreSQL through internal Security Group rules.

If a job fails repeatedly, the message is moved to a Dead Letter Queue for investigation or later reprocessing. CloudWatch collects logs/metrics and triggers alarms when abnormal conditions occur. SNS sends email alerts to Ops/Admin. CloudTrail, AWS Config, VPC Flow Logs, and S3 support audit evidence after deployment.

The entire infrastructure is defined with Terraform. Before deployment, the Terraform plan is checked by an OPA/Rego policy gate to reduce unsafe configurations.

#### Benefits and value

The platform provides the following value:

- reduces public exposure risk for workers and databases;
- enables asynchronous job processing through SQS;
- reduces the risk of losing jobs when the worker is temporarily unavailable;
- supports failed-job handling through Dead Letter Queue;
- provides monitoring and alerting through CloudWatch/SNS;
- generates audit evidence through CloudTrail, AWS Config, VPC Flow Logs, and S3;
- controls infrastructure through Terraform instead of manual console operations;
- remains cost-optimized for lab usage while still being extensible through IaC.

With an estimated 24/7 monthly cost of approximately **85–95 USD**, this is not merely a collection of AWS services. It is a reusable baseline for internal workloads that require private networking, monitoring, alerting, and audit evidence.

---

### 3. Solution Architecture

![Secure Internal Job Processing Platform Architecture](/fcj-workshop/images/2-Proposal/private-by-default/architecture.png)

**Figure:** Architecture diagram of the Secure Internal Job Processing Platform on AWS.

The actual lab deployment uses one EC2 Worker and one Single-AZ RDS PostgreSQL instance. Dưới đây là phiên bản tiếng Anh chuẩn văn phong báo cáo kỹ thuật (Technical Specification/Whitepaper) để bạn đưa vào tài liệu hoặc slide:

"In terms of infrastructure, the system is implemented using a Lean Architecture consisting of a single EC2 Worker and an Amazon RDS PostgreSQL Single-AZ instance to optimize operational costs (FinOps) during the pilot deployment phase. The architecture diagram reflects exactly 100% of the actual provisioned resources.

Despite operating on a Single-AZ configuration, the system still guarantees data integrity and flexible scalability through two core mechanisms:

The Single-AZ design does not mean the system is weak. It is protected by two important mechanisms:

- **Decoupling with SQS:** EventBridge sends jobs to SQS, while the EC2 Worker polls jobs from the queue. If the worker temporarily stops or restarts, unprocessed jobs can remain in SQS according to the configured retention period.
- **Scalability through Terraform:** The infrastructure is modularized using Terraform. When higher availability is required, the system can be extended by adding more workers or enabling Multi-AZ for RDS through IaC changes instead of manual AWS Console operations.

In other words, the project does not attempt to prove high availability by deploying expensive resources in the lab. It demonstrates how to build a cost-controlled internal job-processing platform with queue protection, monitoring, alerting, audit evidence, and IaC-based extensibility.

---

### 4. AWS Services Used

| Service / Component | Role in this project |
|---|---|
| Amazon VPC | Provides network isolation for the system and places EC2/RDS inside private subnets. |
| Amazon EC2 | Runs the internal worker that processes batch jobs from SQS. |
| Amazon RDS PostgreSQL | Stores business data or processing metadata in a private database tier. |
| Amazon SQS | Provides asynchronous job queuing between EventBridge and EC2 Worker; supports retry and Dead Letter Queue. |
| Amazon EventBridge | Creates scheduled business jobs and sends messages to SQS. |
| Amazon CloudWatch | Collects logs/metrics and triggers alarms for abnormal conditions. |
| Amazon SNS | Sends email alerts to Ops/Admin when CloudWatch Alarm is triggered. |
| Amazon S3 | Stores data, logs, or evidence for later review and audit. |
| AWS Systems Manager | Enables EC2 administration through Session Manager instead of public SSH. |
| AWS IAM | Controls access using least privilege. |
| AWS KMS | Encrypts protected storage, including RDS storage. |
| AWS Config | Tracks AWS resource configuration changes over time. |
| AWS CloudTrail | Records API activity for audit and incident investigation. |
| VPC Flow Logs | Sends VPC network traffic metadata to CloudWatch for troubleshooting and audit. |
| VPC Endpoints | Allows EC2 to access required AWS services through private AWS network paths. |
| Terraform | Defines and deploys infrastructure as code. |
| OPA/Rego | Validates Terraform plans before deployment. |

---

### 5. Component Design

#### Compute Layer

The EC2 Worker is deployed inside a private subnet, has no public IP address, and does not expose administrative ports such as SSH to the Internet. It acts as an internal worker for batch jobs such as transaction reconciliation, scheduled data validation, or business log processing.

Administrators access the EC2 instance through AWS Systems Manager Session Manager instead of public SSH. This reduces the external attack surface and supports the project’s private-first design.

#### Database Layer

The database layer uses Amazon RDS PostgreSQL in a Single-AZ configuration. The database is not publicly accessible and only accepts internal connections from the EC2 Worker through Security Group rules. RDS storage is encrypted with AWS KMS. CloudWatch alarms monitor CPU and storage capacity.

#### Queue and Job Processing Layer

Amazon EventBridge creates scheduled jobs and sends messages to Amazon SQS. The EC2 Worker polls messages from SQS and processes them. This decouples job producers from workers, avoids synchronous blocking, and allows jobs to wait safely when the worker is temporarily unavailable.

#### Failure Handling

SQS is configured with a Dead Letter Queue. If a message fails repeatedly, it is moved to the DLQ instead of being lost or retried indefinitely. The DLQ allows Ops/Admin to investigate failures, analyze root causes, and reprocess jobs when needed.

#### Monitoring and Alerting

CloudWatch collects logs/metrics and triggers alarms when abnormal conditions occur. SNS sends email alerts to Ops/Admin. VPC Flow Logs sends network traffic metadata to CloudWatch for network visibility, troubleshooting, and audit.

#### Audit and Governance

CloudTrail records API activity. AWS Config tracks resource configuration changes. IAM controls access, KMS supports encryption, and S3 stores data/logs/evidence for post-deployment review. OPA/Rego acts as a policy gate before Terraform apply.

---

### 6. Architecture Workflow

The numbered diagram represents three main flows: deployment flow, business processing flow, and operational flow.

1. Developer modifies Terraform/IaC code.
2. The IaC Pipeline creates a plan and passes it to the Policy Gate.
3. The Policy Gate checks infrastructure configuration before AWS resources are deployed.
4. EventBridge creates scheduled business jobs and sends messages to SQS.
5. The EC2 Worker inside the VPC polls jobs from SQS.
6. The EC2 Worker processes jobs and connects to RDS when data persistence is required.
7. If a message fails repeatedly, SQS moves it to the Dead Letter Queue.
8. VPC Flow Logs sends network traffic metadata to CloudWatch.
9. EC2 accesses CloudWatch through VPC Endpoints to support private monitoring/logging paths.
10. CloudWatch Alarm triggers SNS.
11. SNS sends email alerts to Ops/Admin.
12. CloudTrail, AWS Config, and S3 support audit evidence and post-deployment verification.

---

### 7. Technical Implementation

The project is implemented through the following phases:

1. **Architecture design:** Define the system as a secure internal job-processing platform, not a public website.
2. **Network design:** Create VPC, private subnets, route tables, and security groups.
3. **Compute/database design:** Deploy EC2 Worker and private RDS PostgreSQL.
4. **Messaging design:** Deploy SQS, Dead Letter Queue, and EventBridge.
5. **Monitoring/alerting design:** Configure CloudWatch alarms and SNS email notification.
6. **Audit/security design:** Configure CloudTrail, AWS Config, VPC Flow Logs, KMS, and IAM.
7. **Infrastructure as Code:** Write Terraform modules for each resource group.
8. **Policy as Code:** Use OPA/Rego to validate the Terraform plan before deployment.
9. **Validation:** Test the flow EventBridge → SQS → EC2 → processing log → DLQ/CloudWatch/SNS.
10. **Cleanup:** Destroy resources with Terraform to avoid unnecessary cost.

---

### 8. Roadmap and Milestones

| Phase | Description |
|---|---|
| Phase 1 | Research the problem and define the project as a secure internal job-processing system. |
| Phase 2 | Design the AWS architecture and draw a diagram that reflects the actual deployed resources. |
| Phase 3 | Write Terraform modules for VPC, EC2, RDS, SQS, EventBridge, CloudWatch, SNS, S3, and governance services. |
| Phase 4 | Write OPA/Rego policy gate to validate Terraform plan. |
| Phase 5 | Deploy infrastructure to AWS and verify resources in AWS Console. |
| Phase 6 | Validate the business flow and collect evidence screenshots. |
| Phase 7 | Write the end-to-end workshop and clean up resources to control cost. |

---

### 9. Budget Estimation

The project is designed with cost control in mind.

For a continuous 24/7 deployment, the estimated monthly cost is approximately **85–95 USD/month**. This estimate assumes:

- one small EC2 Worker;
- one Single-AZ RDS PostgreSQL instance;
- required VPC Endpoints;
- low-volume SQS, EventBridge, CloudWatch, and SNS usage;
- small-scale S3 data/log/evidence storage;
- lab-level usage of KMS, CloudTrail, AWS Config, and VPC Flow Logs.

During the actual demo, the cost can be much lower because the infrastructure is deployed temporarily, validated, documented, and destroyed after evidence collection.

The project avoids NAT Gateway and does not assign a public IP to the EC2 Worker. This helps reduce both cloud cost and public exposure risk. VPC Endpoints are used for required AWS service access.

---

### 10. Risk Assessment

| Risk | Impact | Mitigation |
|---|---|---|
| Worker temporarily stops or fails | Jobs are not processed immediately | SQS stores messages before processing; DLQ stores failed jobs. |
| Single-AZ database has limited HA | Availability may be affected during AZ-level issues | Single-AZ is used for lab cost control; Multi-AZ can be enabled for production. |
| Misconfigured security group or public access | Security exposure | Terraform and OPA/Rego reduce unsafe configuration before deployment. |
| AWS cost increases due to forgotten resources | Budget overrun | Terraform destroy and Billing Forecast are used for cleanup and cost tracking. |
| Missing operational evidence | Harder to audit or review | CloudTrail, AWS Config, VPC Flow Logs, CloudWatch, and evidence screenshots are collected. |

---

### 11. Expected Outcomes

After completion, the project is expected to demonstrate that:

- a backend internal job-processing system can be deployed on AWS;
- EC2 Worker and RDS are not directly exposed to the Internet;
- business jobs can be created by EventBridge and queued in SQS;
- EC2 Worker can poll and process jobs from SQS;
- failed jobs can be isolated through Dead Letter Queue;
- CloudWatch/SNS can alert operators when abnormal conditions occur;
- CloudTrail, AWS Config, VPC Flow Logs, CloudWatch, and S3 can support audit evidence;
- Terraform can recreate and clean up infrastructure;
- OPA/Rego can validate infrastructure policy before deployment.

In the long term, this platform can be extended for internal workloads such as financial reconciliation, customer data validation, compliance evidence generation, invoice processing, and scheduled reporting.