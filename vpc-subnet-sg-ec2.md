#### provider.tf
```
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "5.91.0"
    }
  }
}

provider "aws" {
  region                      = "us-east-1"
  skip_credentials_validation = true
  skip_requesting_account_id  = true
  s3_use_path_style = true

endpoints {
    ec2            = "http://aws:4566"
    apigateway     = "http://aws:4566"
    cloudformation = "http://aws:4566"
    cloudwatch     = "http://aws:4566"
    dynamodb       = "http://aws:4566"
    es             = "http://aws:4566"
    firehose       = "http://aws:4566"
    iam            = "http://aws:4566"
    kinesis        = "http://aws:4566"
    lambda         = "http://aws:4566"
    route53        = "http://aws:4566"
    redshift       = "http://aws:4566"
    s3             = "http://aws:4566"
    secretsmanager = "http://aws:4566"
    ses            = "http://aws:4566"
    sns            = "http://aws:4566"
    sqs            = "http://aws:4566"
    ssm            = "http://aws:4566"
    stepfunctions  = "http://aws:4566"
    sts            = "http://aws:4566"
    rds            = "http://aws:4566"
  }
}
```

#### main.tf

```
# Create the VPC
resource "aws_vpc" "xfusion_priv_vpc" {
  cidr_block           = var.KKE_VPC_CIDR
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "xfusion-priv-vpc"
  }
}

# Create the Private Subnet
resource "aws_subnet" "xfusion_priv_subnet" {
  vpc_id                  = aws_vpc.xfusion_priv_vpc.id
  cidr_block              = var.KKE_SUBNET_CIDR
  map_public_ip_on_launch = false # Auto-assign IP option disabled

  tags = {
    Name = "xfusion-priv-subnet"
  }
}

# Create Security Group allowing access ONLY from within the VPC CIDR
resource "aws_security_group" "xfusion_priv_sg" {
  name        = "xfusion-priv-sg"
  description = "Allow inbound traffic strictly from within the VPC CIDR block"
  vpc_id      = aws_vpc.xfusion_priv_vpc.id

  ingress {
    description = "Internal VPC traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = [var.KKE_VPC_CIDR]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "xfusion-priv-sg"
  }
}

# Create the EC2 Instance
resource "aws_instance" "xfusion_priv_ec2" {
  ami           = "ami-0e2c8caa4b6378d8c" # Placeholder Ubuntu 20.04 LTS AMI (Update according to your region)
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.xfusion_priv_subnet.id

  vpc_security_group_ids = [
    aws_security_group.xfusion_priv_sg.id
  ]

  tags = {
    Name = "xfusion-priv-ec2"
  }
}
```

#### variables.tf

```
variable "KKE_VPC_CIDR" {
  type        = string
  description = "The CIDR block for the xfusion-priv-vpc"
  default     = "10.0.0.0/16"
}

variable "KKE_SUBNET_CIDR" {
  type        = string
  description = "The CIDR block for the xfusion-priv-subnet"
  default     = "10.0.1.0/24"
}
```

#### outputs.tf

```
output "KKE_vpc_name" {
  value       = aws_vpc.xfusion_priv_vpc.tags["Name"]
  description = "The name of the VPC."
}

output "KKE_subnet_name" {
  value       = aws_subnet.xfusion_priv_subnet.tags["Name"]
  description = "The name of the subnet."
}

output "KKE_ec2_private" {
  value       = aws_instance.xfusion_priv_ec2.tags["Name"]
  description = "The name of the EC2 instance."
}
```
