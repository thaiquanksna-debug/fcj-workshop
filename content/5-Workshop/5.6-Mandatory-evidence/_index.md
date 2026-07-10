---
title: "Evidence checklist"
date: 2026-07-04
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---


This section is only used to help reviewers understand the deployed result. Evidence does not replace the end-to-end implementation steps in the previous sections. The core workshop flow remains: setup → source generation → Terraform/OPA deployment → business-flow validation → cleanup.

## A. Core end-to-end evidence

The screenshots below are the most important because they prove that the workload has input, processing, failure handling, and alerting.

### 1. Architecture diagram

![Final architecture diagram](/fcj-workshop/images/5-Workshop/private-by-default/00-final-architecture-diagram.png)

  Final architecture of the Private-by-Default AWS Workload Platform, showing deployment controls, private workload VPC, EC2 Private Worker, SQS, DLQ, CloudWatch Alarm, SNS Email, RDS PostgreSQL, and the audit evidence plane.

### 2. SQS Processing Queue receives business jobs

**SQS Processing Queue receives business processing jobs for asynchronous processing by the EC2 Private Worker.**

![SQS poll messages](/fcj-workshop/images/5-Workshop/private-by-default/01-sqs-poll-for-messages.png)

**What does this evidence prove?**

The processing queue is operational and successfully stores business jobs before they are executed by the private worker.

**Business significance**

Instead of representing a simple JSON payload, each message can represent a real enterprise workload such as:

Daily customer CSV validation
Financial transaction reconciliation
Compliance evidence generation
Invoice processing
Internal reporting

Using Amazon SQS decouples job producers from workers, improves reliability, and enables asynchronous processing without exposing backend systems to the Internet.

### 3. Dead Letter Queue enabled

  The Processing Queue has a Dead Letter Queue enabled to store failed messages after repeated processing failures.

![SQS DLQ enabled](/fcj-workshop/images/5-Workshop/private-by-default/02-sqs-dlq-enabled.png)

**What does this evidence prove?**

The platform supports failure isolation. Messages that repeatedly fail processing are automatically moved to the DLQ instead of being discarded or retried indefinitely.

**Business significance**

In enterprise environments, failed jobs often contain valuable diagnostic information.

Keeping failed jobs inside the DLQ allows operations teams to:

investigate production incidents;
perform root cause analysis;
replay failed jobs after fixes;
prevent endless retry loops that waste compute resources.

This significantly improves operational reliability and auditability.
### 4. Session Manager receives SQS message

  EC2 Private Worker is accessed through AWS Systems Manager Session Manager and receives a message from SQS without SSH or public IP access.

![Session Manager receive message](/fcj-workshop/images/5-Workshop/private-by-default/03-session-manager-receive-message.png)

**What does this evidence prove?**

The worker successfully communicates with AWS managed services through private networking while remaining inaccessible from the public Internet.

**Business significance**

This demonstrates that administrators can securely operate private infrastructure without exposing management ports.

The retrieved message confirms that:

business jobs reach the worker;
private networking functions correctly;
Session Manager replaces traditional SSH administration.

This approach reduces the attack surface while maintaining operational access.

### 5. Worker business flow evidence

  The worker reads a business job from SQS. The message body includes `source = eventbridge_scheduler`, proving the job flow from scheduler/queue to the private worker.

![Worker business flow evidence](/fcj-workshop/images/5-Workshop/private-by-default/04-private-worker-business-flow-evidence.png)

**What does this evidence prove?**

The complete business workflow has started successfully after the worker receives a message from Amazon SQS.

**Business significance**

Although the workshop uses a demo payload, the same workflow can represent production jobs such as:

customer data validation;
financial batch processing;
compliance evidence generation;
scheduled reporting.

This evidence confirms that asynchronous business processing is functioning correctly.

In production environments, similar execution records can later support:

operational troubleshooting;
compliance audits;
SLA verification;
incident investigation.
### 6. Worker process log

  The worker records the process of receiving the job, processing it, and deleting the message from SQS after completion.

![Worker process log](/fcj-workshop/images/5-Workshop/private-by-default/05-worker-process-log.png)

**What does this evidence prove?**

The worker completed processing successfully and produced execution logs that describe each processing stage.

**Business significance**

Application logs are essential operational evidence.

They allow administrators to:

verify successful execution;
investigate processing failures;
reconstruct processing timelines;
demonstrate compliance activities during audits.

These logs become part of the operational evidence generated by the platform.
### 7. CloudWatch Alarm to SNS Email

  CloudWatch Alarm triggers SNS Email to notify the operator. This proves the alerting flow: DLQ metric → CloudWatch Alarm → SNS Email.

![CloudWatch SNS email](/fcj-workshop/images/5-Workshop/private-by-default/06-cloudwatch-alarm-sns-email.png)

**What does this evidence prove?**

Monitoring and alerting are operational.

The platform can automatically notify administrators whenever abnormal conditions occur.

**Business significance**

In production environments, delayed incident detection directly increases business risk.

Automatic notifications enable faster response to:

worker failures;
queue backlogs;
failed processing jobs;
database performance issues.

This improves operational availability and reduces incident response time.
## B. Supporting screenshots for reviewer clarity

The screenshots below are supplementary. They are not required to prove the end-to-end flow, but they help reviewers understand the input source, private compute, private database, and demo cost.

### 8. AWS Billing forecast

  The Billing dashboard shows that the actual demo cost remains low because the infrastructure is deployed temporarily, validated, documented, and cleaned up. The 85–95 USD/month estimate in the proposal refers to a 24/7 pilot deployment.

![Billing forecast](/fcj-workshop/images/5-Workshop/private-by-default/07-billing-forecast-demo-cost.png)

**What does this evidence prove?**

The infrastructure has been successfully deployed and is generating measurable operational costs.

**Business significance**

Cost visibility is an important operational requirement.

The billing forecast allows organizations to:

estimate monthly operating expenses;
compare pilot and production environments;
evaluate infrastructure efficiency;
verify that cleanup procedures successfully stop unnecessary charges.

This demonstrates that the platform considers both technical implementation and operational cost management.
### 9. EventBridge Scheduler enabled

  EventBridge Scheduler is enabled with `rate(5 minutes)` to create demo business processing jobs. The SQS/worker message body provides additional evidence that the job source is `eventbridge_scheduler`.

![EventBridge Scheduler](/fcj-workshop/images/5-Workshop/private-by-default/08-eventbridge-scheduler-enabled.png)

**What does this evidence prove?**

Business workloads can be automatically triggered without manual intervention.

**Business significance**

Many enterprise systems execute scheduled internal jobs such as:

nightly reconciliation;
report generation;
compliance validation;
data synchronization.

EventBridge Scheduler provides a managed scheduling mechanism that automatically creates new processing requests at predefined intervals.
### 10. EC2 Worker has no public IP

  EC2 Private Worker has no public IPv4 address and runs inside a private subnet. Administration is performed through Session Manager instead of public SSH.

![EC2 no public IP](/fcj-workshop/images/5-Workshop/private-by-default/09-ec2-worker-no-public-ip.png)

**What does this evidence prove?**
The application worker is isolated from direct Internet access.

Administrative access is performed exclusively through AWS Systems Manager Session Manager.

**Business significance**

Removing public IP addresses significantly reduces the external attack surface.

This architecture decreases the likelihood of attacks targeting:

SSH services;
brute-force authentication;
Internet-based reconnaissance;
unauthorized remote access.
### 11. RDS Internet access disabled

  Amazon RDS PostgreSQL does not enable Internet access gateway/public access, which keeps the database tier from being directly exposed to the Internet.

![RDS Internet access disabled](/fcj-workshop/images/5-Workshop/private-by-default/10-rds-internet-access-disabled.png)

**What does this evidence prove?**

The production database cannot be accessed directly from the Internet.

Only authorized resources inside the private VPC may establish database connections.

**Business significance**

Sensitive enterprise information remains protected inside the private network.

This architecture helps satisfy common security requirements for organizations handling confidential customer or business data.
### 12. RDS security group restriction

  RDS uses a dedicated security group for the database tier. Inbound access is restricted from the application worker security group instead of exposing the database port to the Internet.

![RDS security group](/fcj-workshop/images/5-Workshop/private-by-default/11-rds-security-group-private-access.png)

**What does this evidence prove?**

Only the application worker security group is permitted to connect to the database.

**Business significance**

Restricting network communication according to the principle of least privilege minimizes lateral movement inside the environment and reduces the impact of compromised resources.
### 13. RDS encryption enabled

  RDS PostgreSQL has encryption enabled for primary storage, proving data-at-rest protection for the database tier.

![RDS encryption enabled](/fcj-workshop/images/5-Workshop/private-by-default/12-rds-encryption-enabled.png)

**What does this evidence prove?**

Data stored inside the database is encrypted at rest using AWS managed encryption.

**Business significance**

Encryption protects sensitive business information even if underlying storage media are compromised.

This configuration supports enterprise security baselines and simplifies future security reviews and compliance assessments.
## Evidence summary

| Evidence group | Question answered |
|---|---|
| SQS + EventBridge | Where does the input/job come from? |
| EC2 Session Manager + worker log | Where and how does the worker process the job? |
| DLQ + CloudWatch + SNS | How does the system handle and alert on failure? |
| EC2 no public IP + RDS private/encrypted | Is the system private-by-default? |
| Billing forecast | Is the demo cost controlled? |
## Reviewer conclusion

The collected evidence demonstrates that the platform successfully achieves the objectives defined in the project proposal.

The screenshots confirm that:

- Business jobs are automatically generated by EventBridge Scheduler.
- Jobs are delivered through Amazon SQS and processed by a private EC2 Worker.
- Failed jobs can be routed to a Dead Letter Queue for investigation.
- CloudWatch Alarms and SNS Email notifications provide operational visibility and incident alerting.
- The workload operates entirely inside private networking boundaries without Internet Gateway, NAT Gateway, bastion hosts, or public IP addresses.
- Database and storage resources use encryption at rest through customer-managed AWS KMS keys.
- Administrative access is performed through AWS Systems Manager Session Manager.

Together, these results validate the Private-by-Default AWS Workload Platform architecture and demonstrate that a secure, auditable, and operationally manageable workload can be deployed on AWS without exposing infrastructure directly to the public Internet.