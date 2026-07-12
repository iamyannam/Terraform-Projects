#### main.tf

```
# Configure the AWS Provider
provider "aws" {
  region = "us-east-1" # Update this to your target region if different
}

# 1. Create the xfusion-ec2 Instance
resource "aws_instance" "xfusion_ec2" {
  ami           = "ami-0c02fb55956c7d316" # Provided Ubuntu AMI
  instance_type = "t2.micro"              # Choose an appropriate instance type

  tags = {
    Name = "xfusion-ec2"
  }
}

# 2. Create the SNS Topic for Alarm Notifications
resource "aws_sns_topic" "xfusion_sns_topic" {
  name = "xfusion-sns-topic"
}

# 3. Create the CloudWatch Alarm
resource "aws_cloudwatch_metric_alarm" "xfusion_alarm" {
  alarm_name          = "xfusion-alarm"
  comparison_operator = "GreaterThanOrEqualToThreshold"
  evaluation_periods  = 1
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 300 # 5 minutes in seconds
  statistic           = "Average"
  threshold           = 90

  dimensions = {
    InstanceId = aws_instance.xfusion_ec2.id
  }

  alarm_description = "This metric monitors EC2 CPU utilization"
  alarm_actions     = [aws_sns_topic.xfusion_sns_topic.arn]
}
```

#### outputs.tf

```
# Output the EC2 instance name tag
output "KKE_instance_name" {
  description = "The name of the EC2 instance"
  value       = aws_instance.xfusion_ec2.tags["Name"]
}

# Output the CloudWatch alarm name
output "KKE_alarm_name" {
  description = "The name of the CloudWatch alarm"
  value       = aws_cloudwatch_metric_alarm.xfusion_alarm.alarm_name
}
```
