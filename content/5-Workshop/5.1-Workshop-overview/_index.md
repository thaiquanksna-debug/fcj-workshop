---
title: "Workshop Overview"
date: 2026-07-04
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Workshop Objective

This workshop presents the actual deployment process of **Secure Internal Job Processing Platform on AWS**, a backend system designed to process internal business jobs in a private-first AWS environment.

The objective of this workshop is not to build a public website or a user-facing application. Instead, it demonstrates that an internal backend workload can be deployed on AWS with the following requirements:

- the worker and database are not exposed to the public Internet;
- jobs are processed through a queue instead of direct public access;
- failed jobs are isolated using a Dead Letter Queue;
- the system provides monitoring and alerting;
- audit evidence is generated for post-deployment review;
- the entire infrastructure is deployed using Terraform;
- infrastructure configuration is validated before deployment using OPA/Rego.

In other words, this workshop demonstrates how a cloud security design from the proposal can be transformed into real AWS infrastructure that is deployable, verifiable, and auditable.

---

## Actual Deployment Scope

The lab version deployed in this workshop uses a cost-optimized configuration consisting of:

- 1 EC2 Worker deployed in a private subnet;
- 1 Amazon RDS PostgreSQL Single-AZ instance;
- 1 Amazon SQS Processing Queue;
- 1 SQS Dead Letter Queue;
- Amazon EventBridge for scheduled job generation;
- Amazon CloudWatch for logs, metrics, and alarms;
- Amazon SNS for email notifications;
- CloudTrail, AWS Config, VPC Flow Logs, and S3 for audit evidence support;
- VPC Endpoints to support private access to required AWS services;
- AWS Systems Manager Session Manager for EC2 administration instead of public SSH.

This scope follows the project principle:

```text id="actual-deployment-principle"
Only describe and prove what is actually deployed.