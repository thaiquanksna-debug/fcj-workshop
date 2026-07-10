---
title: "Cài đặt môi trường cục bộ và quyền truy cập AWS IAM"
date: 2026-07-04
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Mục tiêu

Mục này chuẩn bị máy tính Windows, VS Code, cấu hình AWS CLI SSO và tập hợp quyền (permission set) AWS cần thiết để triển khai workshop. Người đọc bắt đầu mà không có mã nguồn cục bộ.

{{% notice warning %}}
Chạy các câu lệnh Windows trong **Windows PowerShell**. Sau đó, khi workshop mở EC2 thông qua Session Manager, các câu lệnh được đánh dấu là **EC2 Linux terminal** phải được chạy bên trong terminal trình duyệt đó. Không trộn lẫn dấu backtick (`) của PowerShell với dấu gạch chéo ngược (\) của Linux shell.
{{% /notice %}}

# 1. Cài đặt các công cụ cục bộ

Mở **Windows PowerShell với quyền Administrator** và cài đặt các công cụ được yêu cầu:

~~~powershell
winget install --id Microsoft.VisualStudioCode -e
winget install --id Git.Git -e
winget install --id Amazon.AWSCLI -e
winget install --id Hashicorp.Terraform -e
~~~

Cài đặt OPA bằng file binary Windows cố định:

~~~powershell
New-Item -ItemType Directory -Force C:\Tools\opa | Out-Null
Invoke-WebRequest -Uri "https://github.com/open-policy-agent/opa/releases/download/v1.7.1/opa_windows_amd64.exe" -OutFile "C:\Tools\opa\opa.exe"
$CurrentUserPath = [Environment]::GetEnvironmentVariable("Path", "User")
if ($CurrentUserPath -notlike "*C:\Tools\opa*") {{
  [Environment]::SetEnvironmentVariable("Path", "$CurrentUserPath;C:\Tools\opa", "User")
}}
Write-Host "Close this PowerShell window and open a new PowerShell window before checking opa version."
~~~

Đóng PowerShell, mở một cửa sổ PowerShell mới, sau đó kiểm tra lại:

~~~powershell
git --version
aws --version
terraform version
opa version
code --version
~~~

Kết quả mong đợi: mọi câu lệnh đều in ra thông tin phiên bản. Nếu có câu lệnh nào không tìm thấy, hãy cài đặt lại công cụ đó trước khi tiếp tục.

# 2. Tạo người dùng AWS IAM Identity Center

Mở AWS Console bằng tài khoản administrator (quản trị viên).

1. Tìm kiếm **IAM Identity Center**.
2. Mở **IAM Identity Center**.
![Photo](/images/5-Workshop/private-by-default/screen2,1.png)

3. Nếu IAM Identity Center chưa được bật, nhấp vào **Enable**.
![Photo](/images/5-Workshop/private-by-default/screen2,2.jpg)
![Photo](/images/5-Workshop/private-by-default/screen2,3.jpg)

4. Đi tới mục **Users**.
5. Nhấp vào **Add user**.
![Photo](/images/5-Workshop/private-by-default/screen2,4.jpg)
6. Tên người dùng (User name): `mvp-builder`.
7. Email: sử dụng email sẽ dùng để đăng nhập vào AWS.
8. Hoàn tất việc tạo người dùng.
![Photo](/images/5-Workshop/private-by-default/screen2,5.jpg)

# 3. Tạo tập hợp quyền (Permission set)

1. Trong **IAM Identity Center**, đi tới mục **Permission sets**.
![Photo](/images/5-Workshop/private-by-default/screen2,6.png)

2. Nhấp vào **Create permission set**.
![Photo](/images/5-Workshop/private-by-default/screen2,7.png)

3. Chọn **Custom permission set**.
![Photo](/images/5-Workshop/private-by-default/screen2,8.jpg)
4. Tên tập hợp quyền (Permission set name): `PrivateWorkloadTerraformOperator`.
5. Thời gian phiên làm việc (Session duration): chọn 4 giờ hoặc 8 giờ.
6. Tạo tập hợp quyền.
![Photo](/images/5-Workshop/private-by-default/screen2,9.png)
7. Mở `PrivateWorkloadTerraformOperator`.
8. Đi tới **Permissions Set và nhấp vào MVPTerraformOperator**.
9. Nhấp vào **Add inline policy** hoặc **Edit inline policy**.
![Photo](/images/5-Workshop/private-by-default/screen2,10.jpg)

10. Chọn tab **JSON**.
11. Xóa bất kỳ đoạn JSON giữ chỗ (placeholder) nào đang có sẵn.
12. Dán toàn bộ chính sách (policy) bên dưới vào.
Chính sách inline đầy đủ:

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

13. Nhấp Create hoặc Save changes để lưu lại.
![Photo](/images/5-Workshop/private-by-default/screen2,11.jpg)

# 4. Gán người dùng vào tài khoản AWS

1. Trong **IAM Identity Center**, đi tới mục **AWS accounts**.
2. Chọn tài khoản AWS mục tiêu.
3. Nhấp **Assign users or groups**.
![Photo](/images/5-Workshop/private-by-default/screen4,1.png)

4. Chọn người dùng `mvp-builder`.
![Photo](/images/5-Workshop/private-by-default/screen4,2.png)
5. Chọn tập hợp quyền `PrivateWorkloadTerraformOperator`.
![Photo](/images/5-Workshop/private-by-default/screen4,3.png)
6. Gửi (Submit) yêu cầu gán người dùng.
![Photo](/images/5-Workshop/private-by-default/screen4,4.png)

# 5. Cấu hình AWS CLI SSO profile

Trong IAM Identity Center, mở phần **Settings** (Cài đặt) và sao chép đường dẫn **AWS access portal URL**.

Chạy lệnh này trong Windows PowerShell:

~~~powershell
aws configure sso
~~~

Sử dụng các câu trả lời sau:

~~~text
SSO session name: mvp
SSO start URL: dán URL cổng truy cập AWS (AWS access portal URL) từ IAM Identity Center vào đây
SSO region: us-east-1
SSO registration scopes: nhấn Enter
Account: chọn tài khoản AWS của bạn
Role: PrivateWorkloadTerraformOperator
CLI default client Region: us-east-1
CLI default output format: json
CLI profile name: mvp
~~~

Sau đó kiểm tra lại cấu hình profile:

~~~powershell
aws sso logout
aws sso login --profile mvp --no-browser
aws sts get-caller-identity --profile mvp
~~~

Kết quả mong đợi: câu lệnh in ra ID tài khoản AWS của bạn. Chỉ tiếp tục sau khi bước này chạy thành công.

# 6. Kiểm tra lại môi trường cục bộ trước khi tiếp tục

Trước khi tải mã nguồn về, hãy xác nhận rằng tất cả các công cụ bắt buộc đều hoạt động bình thường.

Chạy các lệnh:

git --version
aws --version
terraform version
opa version
code --version

Kết quả mong đợi:

git version x.x.x
aws-cli/x.x.x
Terraform v1.x.x
Version: x.x.x
1.xx.x

Nếu có bất kỳ câu lệnh nào trả về lỗi:

command not found

hoặc

is not recognized as an internal or external command

Hãy đóng PowerShell, mở lại và kiểm tra xem công cụ đó đã được cài đặt chính xác chưa.

# 7. Region AWS khuyến nghị

Workshop này đã được kiểm tra và thử nghiệm tại region:

us-east-1 (N. Virginia)

Việc sử dụng một region khác có thể yêu cầu bạn phải thay đổi:

AMI IDs
Khả năng đáp ứng của dịch vụ (Service availability)
Ước tính chi phí (Pricing estimates)
Tên dịch vụ VPC Endpoint (VPC Endpoint service names)

Để đảm bảo tính nhất quán và dễ tái hiện, hãy sử dụng:

us-east-1

xuyên suốt workshop này.

# 8. Ước tính chi phí triển khai

Dự án này được thiết kế có chủ đích như một mô hình trình diễn cơ sở hạ tầng bảo mật với chi phí thấp.

Các dịch vụ chính cấu thành chi phí:

Dịch vụ | Chi phí ước tính hàng tháng
EC2 Private Worker | 8–12 USD
RDS PostgreSQL | 12–20 USD
VPC Endpoints | 45–60 USD
CloudWatch / SNS / SQS | 1–3 USD
CloudTrail / Config / Logs | 2–5 USD

Tổng chi phí ước tính:

85–95 USD/tháng

khi chạy liên tục 24/7.

Chi phí sử dụng thực tế cho workshop thường thấp hơn nhiều vì môi trường này chỉ chạy trong một khoảng thời gian giới hạn.

Nhiều học viên nhận thấy chỉ tiêu tốn một vài AWS credit vì:

Hệ thống chỉ được triển khai trong thời gian ngắn
Có thể áp dụng các hạn mức của Free Tier (Tầng miễn phí)
Các tài nguyên được xóa bỏ ngay sau khi thử nghiệm xong

# 9. Tiêu chí thành công

Vào cuối workshop, người đọc cần xác nhận được các điều sau:

Bảo mật

✓ Worker không có IP công khai (public IP)

✓ Cơ sở dữ liệu không có endpoint công khai (public endpoint)

✓ Không sử dụng SSH

✓ Truy cập qua Session Manager hoạt động bình thường

✓ Lưu lượng truy cập đi qua VPC Endpoints

Cơ sở hạ tầng

✓ Lệnh triển khai Terraform (deploy) thành công

✓ Lệnh hủy tài nguyên Terraform (destroy) thành công

✓ Xác thực chính sách OPA (Open Policy Agent) thành công

Luồng nghiệp vụ (Business Flow)

✓ Tin nhắn gửi đến được SQS

✓ Worker nhận được tin nhắn từ hàng đợi

✓ Worker xử lý thành công công việc

✓ CloudWatch nhận được log

✓ Tin nhắn lỗi được chuyển đến DLQ (Dead-Letter Queue)

✓ Thông báo qua email của SNS hoạt động bình thường

Kiểm toán (Audit)

✓ CloudTrail ghi lại đầy đủ hoạt động

✓ AWS Config ghi lại trạng thái tài nguyên

✓ VPC Flow Logs tồn tại dữ liệu

✓ Thu thập đầy đủ ảnh chụp màn hình làm bằng chứng

# 10. Trước khi chuyển sang phần tiếp theo

Xác nhận:

[ ] Đã cài đặt VS Code

[ ] Đã cài đặt Git

[ ] Đã cài đặt AWS CLI

[ ] Đã cài đặt Terraform

[ ] Đã cài đặt OPA

[ ] Đã tạo người dùng IAM Identity Center

[ ] Đã gán tập hợp quyền (Permission set)

[ ] Đăng nhập AWS SSO thành công

[ ] Lệnh `aws sts get-caller-identity` hoạt động bình thường

Chỉ tiếp tục sang Mục 5.3 sau khi tất cả các mục trên đã được hoàn thành.