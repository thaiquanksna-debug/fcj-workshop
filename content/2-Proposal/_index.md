---
title: "Proposal"
date: 2026-07-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

## Secure Internal Job Processing Platform on AWS

### 1. Executive Summary

The Secure Internal Job Processing Platform on AWS is a backend platform designed for internal data-processing tasks that must operate in a private, controlled, and auditable cloud environment. It is suitable for workloads such as data reconciliation, business file validation, batch log processing, audit evidence generation, and scheduled compliance activities.

This project is not a public website and does not receive jobs through a public API. Jobs are created on a schedule, placed in a queue, and actively retrieved by an EC2 Worker for processing. Neither the worker nor the database is directly exposed to the Internet. Access to the required AWS services is provided through the VPC Endpoints layer.

The architecture is implemented as a Lean MVP to balance security, verifiability, and cost. It uses one EC2 Worker and one Single-AZ RDS for PostgreSQL instance, while Amazon SQS decouples job creation from job processing. The entire infrastructure is managed with Terraform and evaluated by OPA/Rego before deployment.

---

### 2. Problem Statement

#### Current Problem

Many organizations have internal workloads that are not suitable for deployment as public services, such as:

- reconciling or validating transaction data;
- validating customer data files;
- processing business logs;
- generating reports and audit evidence;
- running scheduled compliance tasks;
- synchronizing or validating data between internal systems.

A typical scenario is an organization periodically receiving CSV files containing customer or transaction data from an internal system. These files must be checked to verify that they follow the expected format, contain all required fields, and are suitable for further processing by downstream systems. This type of workload does not require a public API and should not be executed on a worker that is directly accessible from the Internet.

If these workloads are handled manually or deployed in an insufficiently controlled public environment, organizations may face risks such as exposed workers and databases, interrupted jobs when a worker fails, missing mechanisms for retaining and handling failed messages, lack of operational alerts, insufficient audit evidence, and difficulty controlling infrastructure configuration over time.

#### Proposed Solution

The project proposes a private-first and queue-based job-processing platform. Amazon EventBridge Scheduler creates scheduled jobs and sends messages to Amazon SQS. An EC2 Worker located in a private subnet retrieves jobs from the queue, performs the corresponding processing task, and connects to Amazon RDS for PostgreSQL when data must be stored or retrieved.

In the demonstration scenario, a message may represent a customer CSV validation request and contain information such as the job identifier, job type, job source, and file name. Amazon SQS acts as an intermediary between the job producer and the Worker, allowing the Worker to operate without receiving requests through a public API.

If the Worker is temporarily unavailable, messages remain in the queue and can be processed after the Worker is restored. If a job fails repeatedly, the message is moved to a Dead Letter Queue for isolation and investigation.

Amazon CloudWatch monitors the system state. When messages appear in the Dead Letter Queue, a CloudWatch Alarm can trigger Amazon SNS to notify the Ops/Admin team. AWS CloudTrail, AWS Config, VPC Flow Logs, and Amazon S3 provide data for monitoring, configuration review, and audit evidence storage.

The entire infrastructure is defined using Terraform. Before deployment, the Terraform plan is evaluated by OPA/Rego to detect configurations that violate the private-first security objective, such as an EC2 instance with a public IP address, a publicly accessible RDS instance, a security group exposing administrative ports to the Internet, or unencrypted storage resources.

#### Value Provided

- reduces public exposure of the Worker and database;
- removes the need for a public API for internal processing tasks;
- decouples the job producer and Worker through asynchronous messaging;
- retains jobs when the Worker is temporarily unavailable;
- isolates failed jobs using a Dead Letter Queue;
- supports monitoring, alerting, and audit evidence;
- manages infrastructure consistently through Infrastructure as Code;
- validates security requirements before deployment through Policy as Code;
- provides a reusable security baseline that can be extended as operational requirements grow.

---

### 3. Solution Architecture

![Secure Internal Job Processing Platform Architecture](/fcj-workshop/images/2-Proposal/private-by-default/diagram.jpg)

**Figure:** High-level architecture of the Secure Internal Job Processing Platform on AWS.

The architecture is organized into four main functional areas:

1. **External Deployment Controls:** The Developer, Terraform Pipeline, and OPA/Rego Policy Gate manage infrastructure changes.
2. **Private Workload in the VPC:** The EC2 Worker and RDS for PostgreSQL are deployed in private subnets and do not accept direct Internet access.
3. **Job Processing & Operations:** EventBridge Scheduler, SQS, CloudWatch, SNS, and Session Manager support job creation, asynchronous processing, alerting, and administration.
4. **Security & Governance:** IAM, KMS, Parameter Store, CloudTrail, AWS Config, VPC Flow Logs, and S3 support access control, data protection, and audit evidence retention.

RDS is deployed in Single-AZ mode to fit the MVP scope. Two private database subnets in separate Availability Zones form the DB Subnet Group, but the platform runs only one active database instance and does not claim Multi-AZ failover capability.

The VPC Endpoints layer in the diagram represents private access to the required AWS managed services. Individual endpoint types, route tables, security groups, and policies are defined in Terraform rather than expanded in the high-level diagram.

---

### 4. AWS Services and Components

| Service / Component | Role in the project |
|---|---|
| Amazon VPC | Isolates the workload and provides private subnets for compute, database, and private service access. |
| Amazon EC2 | Runs the worker that processes internal jobs retrieved from SQS. |
| Amazon RDS for PostgreSQL | Stores business data and processing metadata. |
| Amazon SQS | Decouples the Scheduler and Worker and provides retry and DLQ capabilities. |
| Amazon EventBridge Scheduler | Creates scheduled jobs and sends them to the processing queue. |
| Amazon CloudWatch | Collects telemetry, monitors the platform, and evaluates alarms. |
| Amazon SNS | Sends operational notifications to the Ops/Admin team. |
| Amazon S3 | Stores logs and audit evidence. |
| AWS Systems Manager Session Manager | Provides EC2 administration without public SSH access. |
| AWS IAM | Grants permissions to users, workloads, and AWS services. |
| AWS KMS | Protects application data and logs/audit data. |
| AWS Systems Manager Parameter Store | Stores the protected database secret. |
| AWS CloudTrail | Records API activity in the AWS account. |
| AWS Config | Tracks resource configuration and configuration changes. |
| VPC Flow Logs | Records network traffic metadata for visibility and investigation. |
| VPC Endpoints | Provide private connectivity to the required AWS services. |
| Terraform | Defines, deploys, and removes the infrastructure. |
| OPA/Rego | Evaluates policies against the Terraform plan before deployment. |

---

### 5. Component Design

#### Compute and Database Layer

The EC2 Worker operates in the private application subnet and is not directly accessible from the Internet. It uses an IAM role to interact with approved AWS services. RDS for PostgreSQL operates in the private database layer and accepts only the appropriate internal workload connections.

Single-AZ deployment is a cost-optimization decision for the MVP stage. If availability requirements increase, the architecture can be extended through Terraform without redesigning the core platform.

#### Queue and Job-Processing Layer

EventBridge Scheduler acts as the producer, while the EC2 Worker acts as the consumer. Amazon SQS separates these responsibilities, retains jobs while the worker is unavailable, and supports retries. The Dead-Letter Queue isolates messages that cannot be processed successfully so that Ops/Admin can investigate or redrive them safely.

#### Monitoring and Operations Layer

CloudWatch receives workload and network telemetry. The DLQ Alarm monitors failed jobs and triggers SNS when operational notification is required. Session Manager provides a private EC2 administration channel without opening public management ports.

#### Security and Governance Layer

IAM controls access, KMS protects data, and Parameter Store holds sensitive information. CloudTrail and AWS Config provide an audit trail of activity and configuration, while VPC Flow Logs support network-behavior visibility. Audit data is centralized in Amazon S3 for post-deployment review.

---

### 6. Architecture Flow

The numbered steps in the diagram represent deployment, job-processing, and operational activities.

1. The Developer updates the Terraform source code.
2. The Terraform Pipeline creates a deployment plan and submits it to the OPA/Rego Policy Gate.
3. After the plan passes the policy gate, Terraform deploys or updates the AWS resources.
4. EventBridge Scheduler creates a scheduled job and sends a message to the SQS Queue.
5. The EC2 Worker actively polls the queue, receives a message, and acknowledges successful processing.
6. VPC Endpoints provide private connectivity between the worker and the required AWS managed services.
7. The worker connects to RDS for PostgreSQL when a job needs to read or write data.
8. A message that fails repeatedly is moved from the processing queue to the DLQ.
9. A CloudWatch Alarm monitors the appearance of failed messages in the DLQ.
10. When the alarm is triggered, SNS sends a notification to Ops/Admin.
11. Ops/Admin uses Session Manager to perform EC2 administration without public SSH access.

In addition to the numbered flow, CloudTrail and AWS Config deliver audit evidence to S3, while VPC Flow Logs deliver network telemetry to CloudWatch.

---

### 7. Technical Implementation

The infrastructure is defined as Terraform modules grouped by function, including networking, endpoints, compute, database, messaging, monitoring, logging, and encryption. OPA/Rego evaluates the Terraform plan before deployment to identify changes that conflict with the platform's private-first objectives.

The deployment process consists of three main stages:

1. validate the source code and generate a Terraform plan;
2. evaluate the plan through the policy gate;
3. apply the plan, validate the platform flow, and collect evidence.

Resource configuration, IAM permissions, security-group rules, endpoint definitions, alarm thresholds, and lifecycle settings are maintained in the Terraform source code and are intentionally not repeated in this proposal.

---

### 8. Roadmap and Milestones

| Phase | Scope |
|---|---|
| Phase 1 | Define the internal job-processing problem and private-first requirements. |
| Phase 2 | Design the network boundaries, workload flow, and security controls. |
| Phase 3 | Build the Terraform modules and OPA/Rego policies. |
| Phase 4 | Deploy the MVP environment on AWS. |
| Phase 5 | Validate the job flow, failure handling, monitoring, and audit evidence. |
| Phase 6 | Finalize the workshop, documentation, and infrastructure cleanup. |

---

### 9. Estimated Budget

The project is designed for a low-traffic lab/MVP environment. The estimated cost is approximately **USD 85–95 per month** if the entire infrastructure remains active continuously. The actual cost can be significantly lower when the environment is deployed only for implementation, testing, and evidence collection.

The design prioritizes small resources, a Single-AZ RDS deployment, and an architecture that does not depend on a NAT Gateway. Terraform provides a consistent cleanup process after the lab is completed, reducing the risk of unplanned charges.

Actual costs depend on the selected AWS Region, operating duration, log volume, and service usage.

---

### 10. Risk Assessment

| Risk | Potential impact | Mitigation |
|---|---|---|
| EC2 Worker stops or fails | Jobs are not processed immediately. | SQS retains jobs until the worker becomes available again. |
| Single-AZ RDS disruption | The database can become temporarily unavailable. | The limitation is accepted for the MVP and can be addressed when availability requirements increase. |
| Repeated job-processing failure | Retries may continue or hide a business-processing issue. | The DLQ isolates failed messages, while CloudWatch and SNS notify Ops/Admin. |
| Infrastructure misconfiguration | Private-access or security objectives can be weakened. | Terraform and OPA/Rego evaluate changes before deployment. |
| Insufficient operational evidence | Reviews and post-incident investigation become difficult. | CloudTrail, AWS Config, VPC Flow Logs, CloudWatch, and S3 provide audit evidence. |
| Unexpected AWS costs | The lab can exceed its intended budget. | Costs are monitored and resources are removed through Terraform cleanup. |

---

### 11. Expected Outcomes

After completion, the project is expected to demonstrate that:

- an internal job-processing platform can operate without a public API;
- the EC2 Worker and RDS database are not directly exposed to the Internet;
- scheduled jobs can be queued and processed through a pull-based model;
- SQS and the DLQ can retain, retry, and isolate failed jobs;
- CloudWatch and SNS can support operational detection and notification;
- Session Manager can provide private EC2 administration;
- CloudTrail, AWS Config, VPC Flow Logs, and S3 can produce audit evidence;
- Terraform and OPA/Rego can make the infrastructure repeatable, reviewable, and consistently removable.

In the future, the platform can be extended for workloads such as financial reconciliation, customer data validation, compliance evidence generation, invoice processing, and scheduled reporting without changing its core architectural model.
