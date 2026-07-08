#### main.tf

```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1" # Change to your preferred region
}

# 1. DynamoDB Table with minimal configuration
resource "aws_dynamodb_table" "datacenter_table" {
  name         = var.KKE_TABLE_NAME
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "id"

  attribute {
    name = "id"
    type = "S"
  }
}

# 2. IAM Role (with Assume Role Policy Document)
resource "aws_iam_role" "datacenter_role" {
  name = var.KKE_ROLE_NAME

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "ec2.amazonaws.com" # Adjust service principal if needed (e.g., lambda.amazonaws.com)
        }
      }
    ]
  })
}

# 3. IAM Policy granting strictly read-only access to the specific table
resource "aws_iam_policy" "datacenter_readonly_policy" {
  name        = var.KKE_POLICY_NAME
  description = "Read-only access to the specific DynamoDB table"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "dynamodb:GetItem",
          "dynamodb:Scan",
          "dynamodb:Query"
        ]
        Resource = aws_dynamodb_table.datacenter_table.arn
      }
    ]
  })
}

# 4. Attach the policy to the role
resource "aws_iam_role_policy_attachment" "attach_readonly" {
  role       = aws_iam_role.datacenter_role.name
  policy_arn = aws_iam_policy.datacenter_readonly_policy.arn
}
```

#### variables.tf

```
variable "KKE_TABLE_NAME" {
  type        = string
  description = "The name of the DynamoDB table"
}

variable "KKE_ROLE_NAME" {
  type        = string
  description = "The name of the IAM role"
}

variable "KKE_POLICY_NAME" {
  type        = string
  description = "The name of the IAM policy"
}
```

#### terraform.tfvars

```
KKE_TABLE_NAME  = "datacenter-table"
KKE_ROLE_NAME   = "datacenter-role"
KKE_POLICY_NAME = "datacenter-readonly-policy"
```

#### outputs.tf

```
output "kke_dynamodb_table" {
  value       = aws_dynamodb_table.datacenter_table.name
  description = "The name of the DynamoDB table"
}

output "kke_iam_role_name" {
  value       = aws_iam_role.datacenter_role.name
  description = "The name of the IAM role"
}

output "kke_iam_policy_name" {
  value       = aws_iam_policy.datacenter_readonly_policy.name
  description = "The name of the IAM policy"
}
```
