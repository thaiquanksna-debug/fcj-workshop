---
title: "Deploy with Terraform and OPA"
date: 2026-07-04
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# Goal

This section deploys the complete infrastructure. Apply only after OPA returns an empty deny list: `[]`.

# 1. Login to AWS SSO

Run in Windows PowerShell:

~~~powershell
cd D:\mvp_private_by_default_architecture
aws sso logout
aws sso login --profile mvp --no-browser
aws sts get-caller-identity --profile mvp
~~~

Expected result: the output contains your AWS account ID.

# 2. Initialize and validate Terraform

~~~powershell
cd D:\mvp_private_by_default_architecture\infra\envs\mvp
terraform init
terraform fmt -recursive ..\..\modules
terraform fmt -recursive .
terraform validate
~~~

Expected result:

~~~text
Success! The configuration is valid.
~~~

# 3. Create a Terraform plan

~~~powershell
terraform plan -out tfplan.binary
~~~

Terraform should show resources to create, including VPC, private subnets, security groups, VPC endpoints, KMS, S3, CloudTrail, AWS Config, VPC Flow Logs, RDS, EC2 Worker, SQS, DLQ, EventBridge Scheduler, CloudWatch Alarms, and SNS.

# 4. Export the plan to JSON

~~~powershell
cd D:\mvp_private_by_default_architecture
New-Item -ItemType Directory -Force evidence\m9 | Out-Null
cd D:\mvp_private_by_default_architecture\infra\envs\mvp
terraform show -json tfplan.binary > ..\..\..\evidence\m9\final-plan.json
~~~

# 5. Run the OPA/Rego policy gate

~~~powershell
cd D:\mvp_private_by_default_architecture
opa eval --format pretty --data policy\terraform --input evidence\m9\final-plan.json "data.terraform.deny" > evidence\m9\final-opa.txt
Get-Content evidence\m9\final-opa.txt
~~~

Expected output:

~~~text
[]
~~~

# 6. Apply Terraform

Run only if OPA output is `[]`:

~~~powershell
cd D:\mvp_private_by_default_architecture\infra\envs\mvp
terraform apply tfplan.binary
~~~

When Terraform asks for confirmation, type:

~~~text
yes
~~~

RDS can take 7–20 minutes. Wait until Terraform prints:

~~~text
Apply complete!
~~~

# 7. Confirm the SNS email subscription

Open the email address configured in `terraform.tfvars`. Find the email titled similar to:

~~~text
AWS Notification - Subscription Confirmation
~~~

Click **Confirm subscription**. Without this confirmation, CloudWatch Alarm cannot send SNS Email.

# 8. Save Terraform evidence

~~~powershell
cd D:\mvp_private_by_default_architecture\infra\envs\mvp
terraform output > ..\..\..\evidence\m9\terraform-output.txt
terraform output -json > ..\..\..\evidence\m9\terraform-output.json
terraform state list > ..\..\..\evidence\m9\terraform-state.txt
~~~

# 9. Check Session Manager registration

~~~powershell
aws ssm describe-instance-information --profile mvp --region us-east-1
~~~

Expected result: the output contains the EC2 instance ID from `terraform output`.

If the list is empty, wait 3–5 minutes and run the command again. If it is still empty, reboot the worker:

~~~powershell
$InstanceId = terraform output -raw worker_instance_id
aws ec2 reboot-instances --profile mvp --region us-east-1 --instance-ids $InstanceId
Start-Sleep -Seconds 180
aws ssm describe-instance-information --profile mvp --region us-east-1
~~~

# Common issues and exact fixes

## SSO token expired

Symptom:

```text
No valid credential sources found
InvalidGrantException
```

Fix:

```powershell
cd D:\mvp_private_by_default_architecture
aws sso logout
aws sso login --profile mvp --no-browser
aws sts get-caller-identity --profile mvp
```

## Saved plan is stale

Symptom:

```text
Error: Saved plan is stale
```

Fix: create a new plan, export JSON again, run OPA again, then apply the new plan. Never reuse an old plan after Terraform state changed.

## AccessDenied

Symptom: Terraform says `not authorized to perform ...`.

Fix: go back to section 5.2 and verify that the full inline policy was pasted into the `PrivateWorkloadTerraformOperator` permission set, then run `aws sso logout` and `aws sso login --profile mvp --no-browser` again.

# Deployment Result Summary

After successful deployment, the platform should contain:

- 1 private VPC
- 1 private application subnet
- 2 private database subnets
- 1 EC2 Worker instance without public IP
- 1 PostgreSQL RDS instance deployed in private subnets
- Customer-managed KMS keys for application data and logs
- S3 log archive bucket with KMS encryption
- CloudTrail enabled
- AWS Config enabled
- VPC Flow Logs enabled
- Interface VPC Endpoints:
  - KMS
  - STS
  - CloudWatch Logs
  - SSM
  - SSMMessages
  - EC2Messages
  - CloudWatch Monitoring
  - SQS
- Gateway VPC Endpoint:
  - S3
- SQS Processing Queue
- Dead Letter Queue (DLQ)
- SNS Alert Topic
- EventBridge Scheduler
- CloudWatch Alarms
- Session Manager connectivity

The successful deployment demonstrates that the workload platform can operate without Internet Gateway, NAT Gateway, bastion hosts, or public IP addresses while still maintaining operational access through AWS Systems Manager and private AWS service endpoints.