---
title: "Tạo full Terraform và OPA source code"
date: 2026-07-04
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# Mục tiêu

Phần này tạo toàn bộ source code từ con số 0. Người làm không cần có repo sẵn. Một bootstrap script sẽ tự tạo tất cả Terraform modules, root Terraform files và OPA policies.

# 1. Tạo và mở project folder

Chạy trong Windows PowerShell:

~~~powershell
mkdir D:\mvp_private_by_default_architecture
cd D:\mvp_private_by_default_architecture
code .
~~~

VS Code sẽ mở project folder trống.

# 2. Tạo `bootstrap-source.ps1`

Trong VS Code:

1. Right-click vào project folder trống.
2. Chọn **New File**.
3. Đặt tên file đúng như sau:

~~~text
bootstrap-source.ps1
~~~

4. Paste toàn bộ script bên dưới vào `bootstrap-source.ps1`.
5. Save file.

{{% notice warning %}}
Đường dẫn file phải đúng là `D:\mvp_private_by_default_architecture\bootstrap-source.ps1`. Không tạo file này bên trong thư mục `infra`.
{{% /notice %}}

Full bootstrap source code:

~~~powershell
# bootstrap-source.ps1
# Creates the full Terraform + OPA/Rego project source tree.
# Run from: D:\mvp_private_by_default_architecture
$ErrorActionPreference = "Stop"

@(
  "infra\envs\mvp",
  "infra\modules\app",
  "infra\modules\database",
  "infra\modules\endpoints",
  "infra\modules\kms",
  "infra\modules\logging",
  "infra\modules\messaging",
  "infra\modules\monitoring",
  "infra\modules\network",
  "policy\terraform",
  "evidence\m1",
  "evidence\m8",
  "evidence\m9"
) | ForEach-Object { New-Item -ItemType Directory -Force $_ | Out-Null }

@'
.terraform/
*.tfstate
*.tfstate.*
*.tfplan
*.binary
crash.log
crash.*.log
.terraform.lock.hcl
terraform.tfvars
*.auto.tfvars
*.auto.tfvars.json
.DS_Store
.vscode/
'@ | Out-File -Encoding utf8 .gitignore

@'
module "network" {
  source = "../../modules/network"

  project_name = var.project_name
  environment  = var.environment
  aws_region   = var.aws_region
}

module "endpoints" {
  source = "../../modules/endpoints"

  project_name = var.project_name
  environment  = var.environment
  aws_region   = var.aws_region

  vpc_id                     = module.network.vpc_id
  private_app_subnet_ids     = module.network.private_app_subnet_ids
  private_app_route_table_id = module.network.private_app_route_table_id
  private_db_route_table_id  = module.network.private_db_route_table_id
  sg_vpc_endpoints_id        = module.network.sg_vpc_endpoints_id
}

module "kms" {
  source = "../../modules/kms"

  project_name = var.project_name
  environment  = var.environment
}

module "logging" {
  source = "../../modules/logging"

  project_name     = var.project_name
  environment      = var.environment
  aws_region       = var.aws_region
  vpc_id           = module.network.vpc_id
  logs_kms_key_arn = module.kms.logs_key_arn
}

module "database" {
  source = "../../modules/database"

  project_name          = var.project_name
  environment           = var.environment
  private_db_subnet_ids = module.network.private_db_subnet_ids
  sg_db_id              = module.network.sg_db_id
  kms_key_arn           = module.kms.app_data_key_arn
}

module "app" {
  source = "../../modules/app"

  project_name          = var.project_name
  environment           = var.environment
  private_app_subnet_id = module.network.private_app_subnet_ids[0]
  sg_app_id             = module.network.sg_app_id
  kms_key_arn           = module.kms.app_data_key_arn
  processing_queue_arn  = module.messaging.processing_queue_arn
  processing_queue_url  = module.messaging.processing_queue_url
}

module "monitoring" {
  source = "../../modules/monitoring"

  project_name       = var.project_name
  environment        = var.environment
  worker_instance_id = module.app.worker_instance_id
  db_instance_id     = module.database.db_instance_id
}

module "messaging" {
  source = "../../modules/messaging"

  project_name       = var.project_name
  environment        = var.environment
  notification_email = var.notification_email
}
'@ | Out-File -Encoding utf8 infra\envs\mvp\main.tf

@'
output "vpc_id" {
  value = module.network.vpc_id
}

output "private_app_subnet_ids" {
  value = module.network.private_app_subnet_ids
}

output "private_db_subnet_ids" {
  value = module.network.private_db_subnet_ids
}

output "private_app_route_table_id" {
  value = module.network.private_app_route_table_id
}

output "private_db_route_table_id" {
  value = module.network.private_db_route_table_id
}

output "kms_app_data_key_arn" {
  value = module.kms.app_data_key_arn
}

output "kms_logs_key_arn" {
  value = module.kms.logs_key_arn
}

output "logs_bucket_name" {
  value = module.logging.logs_bucket_name
}

output "cloudtrail_name" {
  value = module.logging.cloudtrail_name
}

output "aws_config_recorder_name" {
  value = module.logging.aws_config_recorder_name
}

output "vpc_flow_log_id" {
  value = module.logging.vpc_flow_log_id
}

output "s3_endpoint_id" {
  value = module.endpoints.s3_endpoint_id
}

output "kms_endpoint_id" {
  value = module.endpoints.kms_endpoint_id
}

output "sts_endpoint_id" {
  value = module.endpoints.sts_endpoint_id
}

output "logs_endpoint_id" {
  value = module.endpoints.logs_endpoint_id
}

output "ssm_endpoint_id" {
  value = module.endpoints.ssm_endpoint_id
}

output "ssmmessages_endpoint_id" {
  value = module.endpoints.ssmmessages_endpoint_id
}

output "ec2messages_endpoint_id" {
  value = module.endpoints.ec2messages_endpoint_id
}

output "monitoring_endpoint_id" {
  value = module.endpoints.monitoring_endpoint_id
}

output "sqs_endpoint_id" {
  value = module.endpoints.sqs_endpoint_id
}

output "db_instance_id" {
  value = module.database.db_instance_id
}

output "db_instance_arn" {
  value = module.database.db_instance_arn
}

output "db_subnet_group_name" {
  value = module.database.db_subnet_group_name
}

output "db_password_parameter_name" {
  value = module.database.db_password_parameter_name
}

output "worker_instance_id" {
  value = module.app.worker_instance_id
}

output "worker_private_ip" {
  value = module.app.worker_private_ip
}

output "worker_role_name" {
  value = module.app.worker_role_name
}

output "worker_cpu_alarm_name" {
  value = module.monitoring.worker_cpu_alarm_name
}

output "rds_cpu_alarm_name" {
  value = module.monitoring.rds_cpu_alarm_name
}

output "rds_free_storage_alarm_name" {
  value = module.monitoring.rds_free_storage_alarm_name
}

output "m9_processing_queue_url" {
  value = module.messaging.processing_queue_url
}

output "m9_processing_queue_arn" {
  value = module.messaging.processing_queue_arn
}

output "m9_dlq_url" {
  value = module.messaging.dlq_url
}

output "m9_dlq_arn" {
  value = module.messaging.dlq_arn
}

output "m9_sns_topic_arn" {
  value = module.messaging.sns_topic_arn
}

output "m9_dlq_alarm_name" {
  value = module.messaging.dlq_alarm_name
}

output "m9_eventbridge_schedule_name" {
  value = module.messaging.eventbridge_schedule_name
}
'@ | Out-File -Encoding utf8 infra\envs\mvp\outputs.tf

@'
provider "aws" {
  region  = var.aws_region
  profile = var.aws_profile

  default_tags {
    tags = {
      Project     = var.project_name
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}
'@ | Out-File -Encoding utf8 infra\envs\mvp\providers.tf

@'
notification_email = "your-email@example.com"
'@ | Out-File -Encoding utf8 infra\envs\mvp\terraform.tfvars.example

@'
variable "project_name" {
  description = "Project name used as a resource prefix."
  type        = string
  default     = "private-by-default-mvp"
}

variable "environment" {
  description = "Environment name used as a resource prefix."
  type        = string
  default     = "stage1"
}

variable "aws_region" {
  description = "AWS region for the workshop."
  type        = string
  default     = "us-east-1"
}

variable "aws_profile" {
  description = "AWS CLI profile used by Terraform."
  type        = string
  default     = "mvp"
}

variable "notification_email" {
  description = "Email address for SNS alert notifications."
  type        = string
}
'@ | Out-File -Encoding utf8 infra\envs\mvp\variables.tf

@'
terraform {
  required_version = ">= 1.6.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }

    random = {
      source  = "hashicorp/random"
      version = "~> 3.6"
    }
  }
}
'@ | Out-File -Encoding utf8 infra\envs\mvp\versions.tf

@'
data "aws_region" "current" {}

data "aws_ami" "amazon_linux_2023" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-2023.*-x86_64"]
  }

  filter {
    name   = "architecture"
    values = ["x86_64"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

locals {
  name_prefix = "${var.project_name}-${var.environment}"
}

resource "aws_iam_role" "worker" {
  name = "${local.name_prefix}-ec2-worker-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          Service = "ec2.amazonaws.com"
        }
        Action = "sts:AssumeRole"
      }
    ]
  })

  tags = {
    Name = "${local.name_prefix}-ec2-worker-role"
  }
}

resource "aws_iam_role_policy_attachment" "ssm_core" {
  role       = aws_iam_role.worker.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore"
}

resource "aws_iam_role_policy" "worker_runtime" {
  name = "${local.name_prefix}-worker-runtime-policy"
  role = aws_iam_role.worker.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "cloudwatch:PutMetricData",
          "logs:CreateLogStream",
          "logs:PutLogEvents",
          "logs:DescribeLogStreams"
        ]
        Resource = "*"
      },
      {
        Effect = "Allow"
        Action = [
          "sqs:GetQueueUrl",
          "sqs:GetQueueAttributes",
          "sqs:ReceiveMessage",
          "sqs:DeleteMessage",
          "sqs:ChangeMessageVisibility"
        ]
        Resource = var.processing_queue_arn
      },
      {
        Effect = "Allow"
        Action = [
          "kms:Decrypt",
          "kms:DescribeKey"
        ]
        Resource = var.kms_key_arn
      }
    ]
  })
}

resource "aws_iam_instance_profile" "worker" {
  name = "${local.name_prefix}-ec2-worker-profile"
  role = aws_iam_role.worker.name

  tags = {
    Name = "${local.name_prefix}-ec2-worker-profile"
  }
}

resource "aws_instance" "worker" {
  ami                         = data.aws_ami.amazon_linux_2023.id
  instance_type               = "t3.micro"
  subnet_id                   = var.private_app_subnet_id
  vpc_security_group_ids      = [var.sg_app_id]
  associate_public_ip_address = false
  iam_instance_profile        = aws_iam_instance_profile.worker.name
  user_data_replace_on_change = true

  user_data = <<-EOF
    #!/bin/bash
    mkdir -p /opt/private-worker
    cat > /opt/private-worker/README.txt <<'TXT'
    Private Worker runtime for the Private-by-Default AWS Workload Platform.
    Queue URL: ${var.processing_queue_url}
    TXT
  EOF

  root_block_device {
    encrypted   = true
    kms_key_id  = var.kms_key_arn
    volume_size = 8
    volume_type = "gp3"
  }

  metadata_options {
    http_endpoint = "enabled"
    http_tokens   = "required"
  }

  tags = {
    Name = "${local.name_prefix}-private-worker"
  }
}
'@ | Out-File -Encoding utf8 infra\modules\app\main.tf

@'
output "worker_instance_id" {
  value = aws_instance.worker.id
}

output "worker_private_ip" {
  value = aws_instance.worker.private_ip
}

output "worker_role_name" {
  value = aws_iam_role.worker.name
}
'@ | Out-File -Encoding utf8 infra\modules\app\outputs.tf

@'
variable "project_name" {
  type = string
}

variable "environment" {
  type = string
}

variable "private_app_subnet_id" {
  type = string
}

variable "sg_app_id" {
  type = string
}

variable "kms_key_arn" {
  type = string
}

variable "processing_queue_arn" {
  type = string
}

variable "processing_queue_url" {
  type = string
}
'@ | Out-File -Encoding utf8 infra\modules\app\variables.tf

@'
locals {
  name_prefix = "${var.project_name}-${var.environment}"
}

resource "random_password" "db" {
  length           = 24
  special          = true
  override_special = "!#$%&*()-_=+[]{}<>:?"
}

resource "aws_ssm_parameter" "db_password" {
  name        = "/mvp/${local.name_prefix}/db/password"
  description = "RDS PostgreSQL password for the private-by-default workload platform."
  type        = "SecureString"
  value       = random_password.db.result
  key_id      = var.kms_key_arn

  tags = {
    Name = "${local.name_prefix}-db-password"
  }
}

resource "aws_db_subnet_group" "main" {
  name       = "${local.name_prefix}-db-subnet-group"
  subnet_ids = var.private_db_subnet_ids

  tags = {
    Name = "${local.name_prefix}-db-subnet-group"
  }
}

resource "aws_db_instance" "postgres" {
  identifier              = "${local.name_prefix}-postgres"
  engine                  = "postgres"
  engine_version          = "16"
  instance_class          = "db.t4g.micro"
  allocated_storage       = 20
  db_name                 = "appdb"
  username                = "appuser"
  password                = random_password.db.result
  db_subnet_group_name    = aws_db_subnet_group.main.name
  vpc_security_group_ids  = [var.sg_db_id]
  publicly_accessible     = false
  storage_encrypted       = true
  kms_key_id              = var.kms_key_arn
  multi_az                = false
  backup_retention_period = 1
  deletion_protection     = false
  skip_final_snapshot     = true
  apply_immediately       = true

  tags = {
    Name = "${local.name_prefix}-postgres"
  }
}
'@ | Out-File -Encoding utf8 infra\modules\database\main.tf

@'
output "db_instance_id" {
  value = aws_db_instance.postgres.id
}

output "db_instance_arn" {
  value = aws_db_instance.postgres.arn
}

output "db_subnet_group_name" {
  value = aws_db_subnet_group.main.name
}

output "db_password_parameter_name" {
  value = aws_ssm_parameter.db_password.name
}
'@ | Out-File -Encoding utf8 infra\modules\database\outputs.tf

@'
variable "project_name" {
  type = string
}

variable "environment" {
  type = string
}

variable "private_db_subnet_ids" {
  type = list(string)
}

variable "sg_db_id" {
  type = string
}

variable "kms_key_arn" {
  type = string
}
'@ | Out-File -Encoding utf8 infra\modules\database\variables.tf

@'
locals {
  name_prefix = "${var.project_name}-${var.environment}"
}

resource "aws_vpc_endpoint" "s3" {
  vpc_id            = var.vpc_id
  service_name      = "com.amazonaws.${var.aws_region}.s3"
  vpc_endpoint_type = "Gateway"

  route_table_ids = [
    var.private_app_route_table_id,
    var.private_db_route_table_id
  ]

  tags = {
    Name = "${local.name_prefix}-s3-gateway-endpoint"
  }
}

resource "aws_vpc_endpoint" "kms" {
  vpc_id              = var.vpc_id
  service_name        = "com.amazonaws.${var.aws_region}.kms"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = var.private_app_subnet_ids
  security_group_ids  = [var.sg_vpc_endpoints_id]
  private_dns_enabled = true

  tags = {
    Name = "${local.name_prefix}-kms-interface-endpoint"
  }
}

resource "aws_vpc_endpoint" "sts" {
  vpc_id              = var.vpc_id
  service_name        = "com.amazonaws.${var.aws_region}.sts"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = var.private_app_subnet_ids
  security_group_ids  = [var.sg_vpc_endpoints_id]
  private_dns_enabled = true

  tags = {
    Name = "${local.name_prefix}-sts-interface-endpoint"
  }
}

resource "aws_vpc_endpoint" "logs" {
  vpc_id              = var.vpc_id
  service_name        = "com.amazonaws.${var.aws_region}.logs"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = var.private_app_subnet_ids
  security_group_ids  = [var.sg_vpc_endpoints_id]
  private_dns_enabled = true

  tags = {
    Name = "${local.name_prefix}-logs-interface-endpoint"
  }
}

resource "aws_vpc_endpoint" "ssm" {
  vpc_id              = var.vpc_id
  service_name        = "com.amazonaws.${var.aws_region}.ssm"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = var.private_app_subnet_ids
  security_group_ids  = [var.sg_vpc_endpoints_id]
  private_dns_enabled = true

  tags = {
    Name = "${local.name_prefix}-ssm-interface-endpoint"
  }
}

resource "aws_vpc_endpoint" "ssmmessages" {
  vpc_id              = var.vpc_id
  service_name        = "com.amazonaws.${var.aws_region}.ssmmessages"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = var.private_app_subnet_ids
  security_group_ids  = [var.sg_vpc_endpoints_id]
  private_dns_enabled = true

  tags = {
    Name = "${local.name_prefix}-ssmmessages-interface-endpoint"
  }
}

resource "aws_vpc_endpoint" "ec2messages" {
  vpc_id              = var.vpc_id
  service_name        = "com.amazonaws.${var.aws_region}.ec2messages"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = var.private_app_subnet_ids
  security_group_ids  = [var.sg_vpc_endpoints_id]
  private_dns_enabled = true

  tags = {
    Name = "${local.name_prefix}-ec2messages-interface-endpoint"
  }
}

resource "aws_vpc_endpoint" "monitoring" {
  vpc_id              = var.vpc_id
  service_name        = "com.amazonaws.${var.aws_region}.monitoring"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = var.private_app_subnet_ids
  security_group_ids  = [var.sg_vpc_endpoints_id]
  private_dns_enabled = true

  tags = {
    Name = "${local.name_prefix}-monitoring-interface-endpoint"
  }
}

resource "aws_vpc_endpoint" "sqs" {
  vpc_id              = var.vpc_id
  service_name        = "com.amazonaws.${var.aws_region}.sqs"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = var.private_app_subnet_ids
  security_group_ids  = [var.sg_vpc_endpoints_id]
  private_dns_enabled = true

  tags = {
    Name = "${local.name_prefix}-sqs-interface-endpoint"
  }
}
'@ | Out-File -Encoding utf8 infra\modules\endpoints\main.tf

@'
output "s3_endpoint_id" {
  value = aws_vpc_endpoint.s3.id
}

output "kms_endpoint_id" {
  value = aws_vpc_endpoint.kms.id
}

output "sts_endpoint_id" {
  value = aws_vpc_endpoint.sts.id
}

output "logs_endpoint_id" {
  value = aws_vpc_endpoint.logs.id
}

output "ssm_endpoint_id" {
  value = aws_vpc_endpoint.ssm.id
}

output "ssmmessages_endpoint_id" {
  value = aws_vpc_endpoint.ssmmessages.id
}

output "ec2messages_endpoint_id" {
  value = aws_vpc_endpoint.ec2messages.id
}

output "monitoring_endpoint_id" {
  value = aws_vpc_endpoint.monitoring.id
}

output "sqs_endpoint_id" {
  value = aws_vpc_endpoint.sqs.id
}
'@ | Out-File -Encoding utf8 infra\modules\endpoints\outputs.tf

@'
variable "project_name" {
  type = string
}

variable "environment" {
  type = string
}

variable "aws_region" {
  type = string
}

variable "vpc_id" {
  type = string
}

variable "private_app_subnet_ids" {
  type = list(string)
}

variable "private_app_route_table_id" {
  type = string
}

variable "private_db_route_table_id" {
  type = string
}

variable "sg_vpc_endpoints_id" {
  type = string
}
'@ | Out-File -Encoding utf8 infra\modules\endpoints\variables.tf

@'
data "aws_caller_identity" "current" {}
data "aws_region" "current" {}

locals {
  name_prefix = "${var.project_name}-${var.environment}"
  account_id  = data.aws_caller_identity.current.account_id
  region      = data.aws_region.current.name
}

data "aws_iam_policy_document" "app_data_key" {
  statement {
    sid    = "EnableAccountRootPermissions"
    effect = "Allow"

    principals {
      type        = "AWS"
      identifiers = ["arn:aws:iam::${local.account_id}:root"]
    }

    actions   = ["kms:*"]
    resources = ["*"]
  }

  statement {
    sid    = "AllowRdsUseOfAppDataKey"
    effect = "Allow"

    principals {
      type        = "Service"
      identifiers = ["rds.amazonaws.com"]
    }

    actions = [
      "kms:Encrypt",
      "kms:Decrypt",
      "kms:ReEncrypt*",
      "kms:GenerateDataKey*",
      "kms:DescribeKey",
      "kms:CreateGrant",
      "kms:ListGrants",
      "kms:RevokeGrant"
    ]

    resources = ["*"]
  }

  statement {
    sid    = "AllowSsmUseOfAppDataKey"
    effect = "Allow"

    principals {
      type        = "Service"
      identifiers = ["ssm.amazonaws.com"]
    }

    actions = [
      "kms:Encrypt",
      "kms:Decrypt",
      "kms:ReEncrypt*",
      "kms:GenerateDataKey*",
      "kms:DescribeKey"
    ]

    resources = ["*"]
  }

  statement {
    sid    = "AllowEc2EbsUseOfAppDataKey"
    effect = "Allow"

    principals {
      type        = "Service"
      identifiers = ["ec2.amazonaws.com"]
    }

    actions = [
      "kms:Encrypt",
      "kms:Decrypt",
      "kms:ReEncrypt*",
      "kms:GenerateDataKey*",
      "kms:DescribeKey",
      "kms:CreateGrant",
      "kms:ListGrants",
      "kms:RevokeGrant"
    ]

    resources = ["*"]
  }
}

data "aws_iam_policy_document" "logs_key" {
  statement {
    sid    = "EnableAccountRootPermissions"
    effect = "Allow"

    principals {
      type        = "AWS"
      identifiers = ["arn:aws:iam::${local.account_id}:root"]
    }

    actions   = ["kms:*"]
    resources = ["*"]
  }

  statement {
    sid    = "AllowCloudWatchLogsUseOfLogsKey"
    effect = "Allow"

    principals {
      type        = "Service"
      identifiers = ["logs.${local.region}.amazonaws.com"]
    }

    actions = [
      "kms:Encrypt",
      "kms:Decrypt",
      "kms:ReEncrypt*",
      "kms:GenerateDataKey*",
      "kms:DescribeKey"
    ]

    resources = ["*"]
  }

  statement {
    sid    = "AllowCloudTrailUseOfLogsKey"
    effect = "Allow"

    principals {
      type        = "Service"
      identifiers = ["cloudtrail.amazonaws.com"]
    }

    actions = [
      "kms:GenerateDataKey*",
      "kms:DescribeKey",
      "kms:Decrypt"
    ]

    resources = ["*"]
  }

  statement {
    sid    = "AllowS3UseOfLogsKey"
    effect = "Allow"

    principals {
      type        = "Service"
      identifiers = ["s3.amazonaws.com"]
    }

    actions = [
      "kms:Encrypt",
      "kms:Decrypt",
      "kms:ReEncrypt*",
      "kms:GenerateDataKey*",
      "kms:DescribeKey"
    ]

    resources = ["*"]
  }

  statement {
    sid    = "AllowConfigUseOfLogsKey"
    effect = "Allow"

    principals {
      type        = "Service"
      identifiers = ["config.amazonaws.com"]
    }

    actions = [
      "kms:Encrypt",
      "kms:Decrypt",
      "kms:ReEncrypt*",
      "kms:GenerateDataKey*",
      "kms:DescribeKey"
    ]

    resources = ["*"]
  }
}

resource "aws_kms_key" "app_data" {
  description             = "KMS key for workload app data, database, EBS, and SSM SecureString."
  deletion_window_in_days = 7
  enable_key_rotation     = true
  policy                  = data.aws_iam_policy_document.app_data_key.json

  tags = {
    Name = "${local.name_prefix}-kms-app-data"
  }
}

resource "aws_kms_alias" "app_data" {
  name          = "alias/${local.name_prefix}-app-data"
  target_key_id = aws_kms_key.app_data.key_id
}

resource "aws_kms_key" "logs" {
  description             = "KMS key for workload logs and audit evidence."
  deletion_window_in_days = 7
  enable_key_rotation     = true
  policy                  = data.aws_iam_policy_document.logs_key.json

  tags = {
    Name = "${local.name_prefix}-kms-logs"
  }
}

resource "aws_kms_alias" "logs" {
  name          = "alias/${local.name_prefix}-logs"
  target_key_id = aws_kms_key.logs.key_id
}
'@ | Out-File -Encoding utf8 infra\modules\kms\main.tf

@'
output "app_data_key_arn" {
  value = aws_kms_key.app_data.arn
}

output "logs_key_arn" {
  value = aws_kms_key.logs.arn
}
'@ | Out-File -Encoding utf8 infra\modules\kms\outputs.tf

@'
variable "project_name" {
  type = string
}

variable "environment" {
  type = string
}
'@ | Out-File -Encoding utf8 infra\modules\kms\variables.tf

@'
data "aws_caller_identity" "current" {}

locals {
  name_prefix = "${var.project_name}-${var.environment}"
  account_id  = data.aws_caller_identity.current.account_id
}

resource "random_id" "bucket_suffix" {
  byte_length = 4
}

resource "aws_s3_bucket" "logs" {
  bucket        = "${local.name_prefix}-logs-${random_id.bucket_suffix.hex}"
  force_destroy = true

  tags = {
    Name = "${local.name_prefix}-logs"
  }
}

resource "aws_s3_bucket_public_access_block" "logs" {
  bucket = aws_s3_bucket.logs.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_versioning" "logs" {
  bucket = aws_s3_bucket.logs.id

  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "logs" {
  bucket = aws_s3_bucket.logs.id

  rule {
    apply_server_side_encryption_by_default {
      kms_master_key_id = var.logs_kms_key_arn
      sse_algorithm     = "aws:kms"
    }
  }
}

data "aws_iam_policy_document" "logs_bucket" {
  statement {
    sid    = "AWSCloudTrailAclCheck"
    effect = "Allow"

    principals {
      type        = "Service"
      identifiers = ["cloudtrail.amazonaws.com"]
    }

    actions   = ["s3:GetBucketAcl"]
    resources = [aws_s3_bucket.logs.arn]
  }

  statement {
    sid    = "AWSCloudTrailWrite"
    effect = "Allow"

    principals {
      type        = "Service"
      identifiers = ["cloudtrail.amazonaws.com"]
    }

    actions = ["s3:PutObject"]
    resources = [
      "${aws_s3_bucket.logs.arn}/AWSLogs/${local.account_id}/*"
    ]

    condition {
      test     = "StringEquals"
      variable = "s3:x-amz-acl"
      values   = ["bucket-owner-full-control"]
    }
  }

  statement {
    sid    = "AWSConfigAclCheck"
    effect = "Allow"

    principals {
      type        = "Service"
      identifiers = ["config.amazonaws.com"]
    }

    actions   = ["s3:GetBucketAcl"]
    resources = [aws_s3_bucket.logs.arn]
  }

  statement {
    sid    = "AWSConfigWrite"
    effect = "Allow"

    principals {
      type        = "Service"
      identifiers = ["config.amazonaws.com"]
    }

    actions = ["s3:PutObject"]
    resources = [
      "${aws_s3_bucket.logs.arn}/AWSLogs/${local.account_id}/Config/*"
    ]

    condition {
      test     = "StringEquals"
      variable = "s3:x-amz-acl"
      values   = ["bucket-owner-full-control"]
    }
  }
}

resource "aws_s3_bucket_policy" "logs" {
  bucket = aws_s3_bucket.logs.id
  policy = data.aws_iam_policy_document.logs_bucket.json
}

resource "aws_cloudwatch_log_group" "vpc_flow_logs" {
  name              = "/aws/vpc-flow-logs/${local.name_prefix}"
  retention_in_days = 14
  kms_key_id        = var.logs_kms_key_arn

  tags = {
    Name = "${local.name_prefix}-vpc-flow-logs"
  }
}

resource "aws_iam_role" "vpc_flow_logs" {
  name = "${local.name_prefix}-vpc-flow-logs-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          Service = "vpc-flow-logs.amazonaws.com"
        }
        Action = "sts:AssumeRole"
      }
    ]
  })

  tags = {
    Name = "${local.name_prefix}-vpc-flow-logs-role"
  }
}

resource "aws_iam_role_policy" "vpc_flow_logs" {
  name = "${local.name_prefix}-vpc-flow-logs-policy"
  role = aws_iam_role.vpc_flow_logs.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "logs:CreateLogStream",
          "logs:PutLogEvents",
          "logs:DescribeLogGroups",
          "logs:DescribeLogStreams"
        ]
        Resource = "*"
      }
    ]
  })
}

resource "aws_flow_log" "vpc" {
  iam_role_arn    = aws_iam_role.vpc_flow_logs.arn
  log_destination = aws_cloudwatch_log_group.vpc_flow_logs.arn
  traffic_type    = "ALL"
  vpc_id          = var.vpc_id

  tags = {
    Name = "${local.name_prefix}-vpc-flow-log"
  }
}

resource "aws_cloudtrail" "main" {
  name                          = "${local.name_prefix}-trail"
  s3_bucket_name                = aws_s3_bucket.logs.id
  kms_key_id                    = var.logs_kms_key_arn
  include_global_service_events = true
  is_multi_region_trail         = false
  enable_logging                = true

  depends_on = [aws_s3_bucket_policy.logs]

  tags = {
    Name = "${local.name_prefix}-trail"
  }
}

resource "aws_iam_role" "config" {
  name = "${local.name_prefix}-config-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          Service = "config.amazonaws.com"
        }
        Action = "sts:AssumeRole"
      }
    ]
  })

  tags = {
    Name = "${local.name_prefix}-config-role"
  }
}

resource "aws_iam_role_policy_attachment" "config" {
  role       = aws_iam_role.config.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWS_ConfigRole"
}

resource "aws_config_configuration_recorder" "main" {
  name     = "${local.name_prefix}-config-recorder"
  role_arn = aws_iam_role.config.arn

  recording_group {
    all_supported                 = true
    include_global_resource_types = false
  }
}

resource "aws_config_delivery_channel" "main" {
  name           = "${local.name_prefix}-config-delivery-channel"
  s3_bucket_name = aws_s3_bucket.logs.id

  depends_on = [aws_config_configuration_recorder.main, aws_s3_bucket_policy.logs]
}

resource "aws_config_configuration_recorder_status" "main" {
  name       = aws_config_configuration_recorder.main.name
  is_enabled = true

  depends_on = [aws_config_delivery_channel.main]
}
'@ | Out-File -Encoding utf8 infra\modules\logging\main.tf

@'
output "logs_bucket_name" {
  value = aws_s3_bucket.logs.bucket
}

output "cloudtrail_name" {
  value = aws_cloudtrail.main.name
}

output "aws_config_recorder_name" {
  value = aws_config_configuration_recorder.main.name
}

output "vpc_flow_log_id" {
  value = aws_flow_log.vpc.id
}
'@ | Out-File -Encoding utf8 infra\modules\logging\outputs.tf

@'
variable "project_name" {
  type = string
}

variable "environment" {
  type = string
}

variable "aws_region" {
  type = string
}

variable "vpc_id" {
  type = string
}

variable "logs_kms_key_arn" {
  type = string
}
'@ | Out-File -Encoding utf8 infra\modules\logging\variables.tf

@'
locals {
  name_prefix = "${var.project_name}-${var.environment}"
}

resource "aws_sqs_queue" "dlq" {
  name                      = "${local.name_prefix}-failed-jobs-dlq"
  message_retention_seconds = 1209600

  tags = {
    Name = "${local.name_prefix}-failed-jobs-dlq"
  }
}

resource "aws_sqs_queue" "processing" {
  name                       = "${local.name_prefix}-processing-queue"
  visibility_timeout_seconds = 60
  message_retention_seconds  = 345600

  redrive_policy = jsonencode({
    deadLetterTargetArn = aws_sqs_queue.dlq.arn
    maxReceiveCount     = 3
  })

  tags = {
    Name = "${local.name_prefix}-processing-queue"
  }
}

resource "aws_sns_topic" "alerts" {
  name = "${local.name_prefix}-alerts"

  tags = {
    Name = "${local.name_prefix}-alerts"
  }
}

resource "aws_sns_topic_subscription" "email" {
  topic_arn = aws_sns_topic.alerts.arn
  protocol  = "email"
  endpoint  = var.notification_email
}

resource "aws_cloudwatch_metric_alarm" "dlq_visible_messages" {
  alarm_name          = "${local.name_prefix}-dlq-visible-messages"
  alarm_description   = "Alert when failed job messages appear in the DLQ."
  namespace           = "AWS/SQS"
  metric_name         = "ApproximateNumberOfMessagesVisible"
  statistic           = "Sum"
  period              = 60
  evaluation_periods  = 1
  threshold           = 0
  comparison_operator = "GreaterThanThreshold"
  treat_missing_data  = "notBreaching"

  dimensions = {
    QueueName = aws_sqs_queue.dlq.name
  }

  alarm_actions = [aws_sns_topic.alerts.arn]

  tags = {
    Name = "${local.name_prefix}-dlq-visible-messages"
  }
}

resource "aws_iam_role" "eventbridge_to_sqs" {
  name = "${local.name_prefix}-eventbridge-to-sqs-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          Service = "scheduler.amazonaws.com"
        }
        Action = "sts:AssumeRole"
      }
    ]
  })

  tags = {
    Name = "${local.name_prefix}-eventbridge-to-sqs-role"
  }
}

resource "aws_iam_role_policy" "eventbridge_to_sqs" {
  name = "${local.name_prefix}-eventbridge-to-sqs-policy"
  role = aws_iam_role.eventbridge_to_sqs.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect   = "Allow"
        Action   = ["sqs:SendMessage"]
        Resource = aws_sqs_queue.processing.arn
      }
    ]
  })
}

resource "aws_scheduler_schedule" "job_every_5_minutes" {
  name        = "${local.name_prefix}-job-every-5-minutes"
  description = "Create demo business processing jobs every 5 minutes."

  flexible_time_window {
    mode = "OFF"
  }

  schedule_expression = "rate(5 minutes)"

  target {
    arn      = aws_sqs_queue.processing.arn
    role_arn = aws_iam_role.eventbridge_to_sqs.arn

    input = jsonencode({
      job_type = "demo_csv_validation"
      priority = "normal"
      source   = "eventbridge_scheduler"
    })
  }
}
'@ | Out-File -Encoding utf8 infra\modules\messaging\main.tf

@'
output "processing_queue_url" {
  value = aws_sqs_queue.processing.url
}

output "processing_queue_arn" {
  value = aws_sqs_queue.processing.arn
}

output "dlq_url" {
  value = aws_sqs_queue.dlq.url
}

output "dlq_arn" {
  value = aws_sqs_queue.dlq.arn
}

output "sns_topic_arn" {
  value = aws_sns_topic.alerts.arn
}

output "dlq_alarm_name" {
  value = aws_cloudwatch_metric_alarm.dlq_visible_messages.alarm_name
}

output "eventbridge_schedule_name" {
  value = aws_scheduler_schedule.job_every_5_minutes.name
}
'@ | Out-File -Encoding utf8 infra\modules\messaging\outputs.tf

@'
variable "project_name" {
  type = string
}

variable "environment" {
  type = string
}

variable "notification_email" {
  type = string
}
'@ | Out-File -Encoding utf8 infra\modules\messaging\variables.tf

@'
locals {
  name_prefix = "${var.project_name}-${var.environment}"
}

resource "aws_cloudwatch_metric_alarm" "worker_cpu_high" {
  alarm_name          = "${local.name_prefix}-ec2-worker-cpu-high"
  alarm_description   = "Alert when EC2 worker CPU is high."
  namespace           = "AWS/EC2"
  metric_name         = "CPUUtilization"
  statistic           = "Average"
  period              = 300
  evaluation_periods  = 1
  threshold           = 80
  comparison_operator = "GreaterThanThreshold"
  treat_missing_data  = "notBreaching"

  dimensions = {
    InstanceId = var.worker_instance_id
  }

  tags = {
    Name = "${local.name_prefix}-ec2-worker-cpu-high"
  }
}

resource "aws_cloudwatch_metric_alarm" "rds_cpu_high" {
  alarm_name          = "${local.name_prefix}-rds-cpu-high"
  alarm_description   = "Alert when RDS CPU is high."
  namespace           = "AWS/RDS"
  metric_name         = "CPUUtilization"
  statistic           = "Average"
  period              = 300
  evaluation_periods  = 1
  threshold           = 80
  comparison_operator = "GreaterThanThreshold"
  treat_missing_data  = "notBreaching"

  dimensions = {
    DBInstanceIdentifier = var.db_instance_id
  }

  tags = {
    Name = "${local.name_prefix}-rds-cpu-high"
  }
}

resource "aws_cloudwatch_metric_alarm" "rds_free_storage_low" {
  alarm_name          = "${local.name_prefix}-rds-free-storage-low"
  alarm_description   = "Alert when RDS free storage is low."
  namespace           = "AWS/RDS"
  metric_name         = "FreeStorageSpace"
  statistic           = "Average"
  period              = 300
  evaluation_periods  = 1
  threshold           = 2147483648
  comparison_operator = "LessThanThreshold"
  treat_missing_data  = "notBreaching"

  dimensions = {
    DBInstanceIdentifier = var.db_instance_id
  }

  tags = {
    Name = "${local.name_prefix}-rds-free-storage-low"
  }
}
'@ | Out-File -Encoding utf8 infra\modules\monitoring\main.tf

@'
output "worker_cpu_alarm_name" {
  value = aws_cloudwatch_metric_alarm.worker_cpu_high.alarm_name
}

output "rds_cpu_alarm_name" {
  value = aws_cloudwatch_metric_alarm.rds_cpu_high.alarm_name
}

output "rds_free_storage_alarm_name" {
  value = aws_cloudwatch_metric_alarm.rds_free_storage_low.alarm_name
}
'@ | Out-File -Encoding utf8 infra\modules\monitoring\outputs.tf

@'
variable "project_name" {
  type = string
}

variable "environment" {
  type = string
}

variable "worker_instance_id" {
  type = string
}

variable "db_instance_id" {
  type = string
}
'@ | Out-File -Encoding utf8 infra\modules\monitoring\variables.tf

@'
data "aws_availability_zones" "available" {
  state = "available"
}

locals {
  name_prefix = "${var.project_name}-${var.environment}"
}

resource "aws_vpc" "main" {
  cidr_block           = "10.10.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "${local.name_prefix}-vpc"
  }
}

resource "aws_subnet" "private_app" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.10.10.0/24"
  availability_zone       = data.aws_availability_zones.available.names[0]
  map_public_ip_on_launch = false

  tags = {
    Name = "${local.name_prefix}-private-app-subnet-a"
    Tier = "app"
  }
}

resource "aws_subnet" "private_db_a" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.10.20.0/24"
  availability_zone       = data.aws_availability_zones.available.names[0]
  map_public_ip_on_launch = false

  tags = {
    Name = "${local.name_prefix}-private-db-subnet-a"
    Tier = "database"
  }
}

resource "aws_subnet" "private_db_b" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.10.21.0/24"
  availability_zone       = data.aws_availability_zones.available.names[1]
  map_public_ip_on_launch = false

  tags = {
    Name = "${local.name_prefix}-private-db-subnet-b"
    Tier = "database"
  }
}

resource "aws_route_table" "private_app" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "${local.name_prefix}-private-app-rt"
  }
}

resource "aws_route_table" "private_db" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "${local.name_prefix}-private-db-rt"
  }
}

resource "aws_route_table_association" "private_app" {
  subnet_id      = aws_subnet.private_app.id
  route_table_id = aws_route_table.private_app.id
}

resource "aws_route_table_association" "private_db_a" {
  subnet_id      = aws_subnet.private_db_a.id
  route_table_id = aws_route_table.private_db.id
}

resource "aws_route_table_association" "private_db_b" {
  subnet_id      = aws_subnet.private_db_b.id
  route_table_id = aws_route_table.private_db.id
}

resource "aws_security_group" "app" {
  name        = "${local.name_prefix}-sg-app"
  description = "Private EC2 worker security group. No inbound SSH/RDP."
  vpc_id      = aws_vpc.main.id

  tags = {
    Name = "${local.name_prefix}-sg-app"
  }
}

resource "aws_security_group" "db" {
  name        = "${local.name_prefix}-sg-db"
  description = "RDS PostgreSQL security group."
  vpc_id      = aws_vpc.main.id

  tags = {
    Name = "${local.name_prefix}-sg-db"
  }
}

resource "aws_security_group" "vpc_endpoints" {
  name        = "${local.name_prefix}-sg-vpc-endpoints"
  description = "Interface endpoint security group."
  vpc_id      = aws_vpc.main.id

  tags = {
    Name = "${local.name_prefix}-sg-vpc-endpoints"
  }
}

resource "aws_security_group_rule" "db_from_app_postgres" {
  type                     = "ingress"
  description              = "Allow PostgreSQL from EC2 worker"
  from_port                = 5432
  to_port                  = 5432
  protocol                 = "tcp"
  security_group_id        = aws_security_group.db.id
  source_security_group_id = aws_security_group.app.id
}

resource "aws_security_group_rule" "app_to_db_postgres" {
  type                     = "egress"
  description              = "Allow EC2 worker to access PostgreSQL"
  from_port                = 5432
  to_port                  = 5432
  protocol                 = "tcp"
  security_group_id        = aws_security_group.app.id
  source_security_group_id = aws_security_group.db.id
}

resource "aws_security_group_rule" "vpc_endpoints_from_app_https" {
  type                     = "ingress"
  description              = "Allow HTTPS from EC2 worker to interface endpoints"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  security_group_id        = aws_security_group.vpc_endpoints.id
  source_security_group_id = aws_security_group.app.id
}

resource "aws_security_group_rule" "app_to_vpc_endpoints_https" {
  type                     = "egress"
  description              = "Allow EC2 worker to access interface endpoints over HTTPS"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  security_group_id        = aws_security_group.app.id
  source_security_group_id = aws_security_group.vpc_endpoints.id
}
'@ | Out-File -Encoding utf8 infra\modules\network\main.tf

@'
output "vpc_id" {
  value = aws_vpc.main.id
}

output "private_app_subnet_ids" {
  value = [aws_subnet.private_app.id]
}

output "private_db_subnet_ids" {
  value = [aws_subnet.private_db_a.id, aws_subnet.private_db_b.id]
}

output "private_app_route_table_id" {
  value = aws_route_table.private_app.id
}

output "private_db_route_table_id" {
  value = aws_route_table.private_db.id
}

output "sg_app_id" {
  value = aws_security_group.app.id
}

output "sg_db_id" {
  value = aws_security_group.db.id
}

output "sg_vpc_endpoints_id" {
  value = aws_security_group.vpc_endpoints.id
}
'@ | Out-File -Encoding utf8 infra\modules\network\outputs.tf

@'
variable "project_name" {
  type = string
}

variable "environment" {
  type = string
}

variable "aws_region" {
  type = string
}
'@ | Out-File -Encoding utf8 infra\modules\network\variables.tf

@'
package terraform

deny contains msg if {
  rc := input.resource_changes[_]
  rc.type == "aws_db_instance"
  after := rc.change.after
  after.publicly_accessible == true
  msg := sprintf("RDS instance must not be publicly accessible: %s", [rc.address])
}

deny contains msg if {
  rc := input.resource_changes[_]
  rc.type == "aws_db_instance"
  after := rc.change.after
  after.storage_encrypted != true
  msg := sprintf("RDS instance must use storage encryption: %s", [rc.address])
}

deny contains msg if {
  rc := input.resource_changes[_]
  rc.type == "aws_db_instance"
  after := rc.change.after
  kms_key_id := object.get(after, "kms_key_id", null)
  kms_key_id == null
  msg := sprintf("RDS instance must use a KMS key: %s", [rc.address])
}

deny contains msg if {
  rc := input.resource_changes[_]
  rc.type == "aws_db_instance"
  after := rc.change.after
  after.multi_az == true
  msg := sprintf("Stage 1 uses Single-AZ RDS for cost control: %s", [rc.address])
}

deny contains msg if {
  rc := input.resource_changes[_]
  rc.type == "aws_ssm_parameter"
  after := rc.change.after
  after.type != "SecureString"
  msg := sprintf("Database password parameter must be SecureString: %s", [rc.address])
}
'@ | Out-File -Encoding utf8 policy\terraform\database_invariants.rego

@'
package terraform

deny contains msg if {
  rc := input.resource_changes[_]
  rc.type == "aws_instance"
  indexof(rc.address, "worker") != -1
  after := rc.change.after
  after.associate_public_ip_address == true
  msg := sprintf("EC2 worker must not associate public IP: %s", [rc.address])
}

deny contains msg if {
  rc := input.resource_changes[_]
  rc.type == "aws_instance"
  indexof(rc.address, "worker") != -1
  after := rc.change.after
  key_name := object.get(after, "key_name", null)
  key_name != null
  key_name != ""
  msg := sprintf("EC2 worker must not use SSH key pair: %s", [rc.address])
}

deny contains msg if {
  rc := input.resource_changes[_]
  rc.type == "aws_instance"
  indexof(rc.address, "worker") != -1
  after := rc.change.after
  profile := object.get(after, "iam_instance_profile", null)
  profile == null
  msg := sprintf("EC2 worker must use IAM instance profile: %s", [rc.address])
}

deny contains msg if {
  rc := input.resource_changes[_]
  rc.type == "aws_instance"
  indexof(rc.address, "worker") != -1
  after := rc.change.after
  metadata_options := object.get(after, "metadata_options", [])
  count(metadata_options) > 0
  metadata_options[0].http_tokens != "required"
  msg := sprintf("EC2 worker must require IMDSv2: %s", [rc.address])
}

deny contains msg if {
  rc := input.resource_changes[_]
  rc.type == "aws_instance"
  indexof(rc.address, "worker") != -1
  after := rc.change.after
  root_block_device := object.get(after, "root_block_device", [])
  count(root_block_device) > 0
  root_block_device[0].encrypted != true
  msg := sprintf("EC2 worker root EBS must be encrypted: %s", [rc.address])
}
'@ | Out-File -Encoding utf8 policy\terraform\ec2_worker_invariants.rego

@'
package terraform

deny contains msg if {
  rc := input.resource_changes[_]
  rc.type == "aws_subnet"
  after := rc.change.after
  after.map_public_ip_on_launch == true
  msg := sprintf("Private subnet must not map public IP on launch: %s", [rc.address])
}

deny contains msg if {
  rc := input.resource_changes[_]
  rc.type == "aws_internet_gateway"
  msg := sprintf("Internet Gateway is not allowed in the private-by-default workload platform: %s", [rc.address])
}

deny contains msg if {
  rc := input.resource_changes[_]
  rc.type == "aws_security_group_rule"
  after := rc.change.after
  after.type == "ingress"
  after.protocol == "tcp"
  after.from_port <= 22
  after.to_port >= 22
  cidr_blocks := object.get(after, "cidr_blocks", [])
  cidr_blocks[_] == "0.0.0.0/0"
  msg := sprintf("Public SSH ingress is not allowed: %s", [rc.address])
}

deny contains msg if {
  rc := input.resource_changes[_]
  rc.type == "aws_security_group_rule"
  after := rc.change.after
  after.type == "ingress"
  after.protocol == "tcp"
  after.from_port <= 3389
  after.to_port >= 3389
  cidr_blocks := object.get(after, "cidr_blocks", [])
  cidr_blocks[_] == "0.0.0.0/0"
  msg := sprintf("Public RDP ingress is not allowed: %s", [rc.address])
}
'@ | Out-File -Encoding utf8 policy\terraform\network_invariants.rego

Write-Host "Source tree created successfully."
~~~

# 3. Chạy bootstrap script

Chạy trong Windows PowerShell:

~~~powershell
cd D:\mvp_private_by_default_architecture
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\bootstrap-source.ps1
~~~

Kết quả đúng:

~~~text
Source tree created successfully.
~~~

# 4. Tạo `terraform.tfvars`

Terraform cần email để tạo SNS alert subscription. Thay email bên dưới bằng email thật sẽ nhận cảnh báo AWS.

Chạy trong Windows PowerShell:

~~~powershell
cd D:\mvp_private_by_default_architecture
@'
notification_email = "your-email@example.com"
'@ | Out-File -Encoding utf8 infra\envs\mvp\terraform.tfvars
~~~

# 5. Kiểm tra source tree đã được tạo

Run:

~~~powershell
cd D:\mvp_private_by_default_architecture
Get-ChildItem infra\envs\mvp
Get-ChildItem infra\modules
Get-ChildItem policy\terraform
~~~

Kết quả đúng:

~~~text
infra\envs\mvp contains versions.tf, providers.tf, variables.tf, main.tf, outputs.tf, terraform.tfvars
infra\modules contains app, database, endpoints, kms, logging, messaging, monitoring, network
policy\terraform contains database_invariants.rego, ec2_worker_invariants.rego, network_invariants.rego
~~~

Root module wiring mà script tạo ra là:

~~~hcl
module "network" {
  source = "../../modules/network"

  project_name = var.project_name
  environment  = var.environment
  aws_region   = var.aws_region
}

module "endpoints" {
  source = "../../modules/endpoints"

  project_name = var.project_name
  environment  = var.environment
  aws_region   = var.aws_region

  vpc_id                     = module.network.vpc_id
  private_app_subnet_ids     = module.network.private_app_subnet_ids
  private_app_route_table_id = module.network.private_app_route_table_id
  private_db_route_table_id  = module.network.private_db_route_table_id
  sg_vpc_endpoints_id        = module.network.sg_vpc_endpoints_id
}

module "kms" {
  source = "../../modules/kms"

  project_name = var.project_name
  environment  = var.environment
}

module "logging" {
  source = "../../modules/logging"

  project_name     = var.project_name
  environment      = var.environment
  aws_region       = var.aws_region
  vpc_id           = module.network.vpc_id
  logs_kms_key_arn = module.kms.logs_key_arn
}

module "database" {
  source = "../../modules/database"

  project_name          = var.project_name
  environment           = var.environment
  private_db_subnet_ids = module.network.private_db_subnet_ids
  sg_db_id              = module.network.sg_db_id
  kms_key_arn           = module.kms.app_data_key_arn
}

module "app" {
  source = "../../modules/app"

  project_name          = var.project_name
  environment           = var.environment
  private_app_subnet_id = module.network.private_app_subnet_ids[0]
  sg_app_id             = module.network.sg_app_id
  kms_key_arn           = module.kms.app_data_key_arn
  processing_queue_arn  = module.messaging.processing_queue_arn
  processing_queue_url  = module.messaging.processing_queue_url
}

module "monitoring" {
  source = "../../modules/monitoring"

  project_name       = var.project_name
  environment        = var.environment
  worker_instance_id = module.app.worker_instance_id
  db_instance_id     = module.database.db_instance_id
}

module "messaging" {
  source = "../../modules/messaging"

  project_name       = var.project_name
  environment        = var.environment
  notification_email = var.notification_email
}
~~~
