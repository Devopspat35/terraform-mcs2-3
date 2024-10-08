# Terraform 10 high level blocks

https://github.com/LandmakTechnology/terraform-master-class-series/blob/main/03%20-%20Language%20Syntax/b).%20top-level-blocks-sample.md

https://registry.terraform.io/namespaces/hashicorp

https://registry.terraform.io/providers/hashicorp/aws/latest


## Block-1: **Terraform Settings Block**
```
terraform {
  required_version = "~> 1.0" #1.1.4/5/6/7   1.2/3/4/5 1.1.4/5/6/7
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```


## Block-2: **Provider Block**
```
provider "aws" {
  profile = "default" # AWS Credentials Profile configured on your local desktop terminal  $HOME/.aws/credentials
  region  = "us-east-2"
}
```

## Block-3: **Resource Block**
```
resource "aws_instance" "inst1" {
  ami           = "ami-0e5b6b6a9f3db6db8" # Amazon Linux
  instance_type = var.instance_type
}
```

## Block-4: **Input Variables Block**
```
variable "instance_type" {
  default     = "t2.micro"
  description = "EC2 Instance Type"
  type        = string
}
```

## Block-5: **Output Values Block**
```
output "ec2_instance_publicip" {
  description = "EC2 Instance Public IP"
  value       = aws_instance.inst1.public_ip
}
```

## Block-6: **Local Values Block**
 - An example to have a bucket name that is a concatenation of an app name and environment name.
```
locals {
  name = "${var.app_name}-${var.environment_name}"
}

bucket_name = local.name
```

## Block-7: **Data sources Block**
 - This example is used to get the latest AMI ID for Amazon Linux2 OS
```
data "aws_ami" "amzLinux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*"]
  }

  filter {
    name   = "root-device-type"
    values = ["ebs"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  filter {
    name   = "architecture"
    values = ["x86_64"]
  }
}
```

## Block-8: **Modules Block**
- An example of an AWS EC2 Instance Module from the terraform registry.
```
module "ec2_cluster" {
  source                 = "terraform-aws-modules/ec2-instance/aws"
  version                = "~> 2.0"

  name                   = "my-modules-demo"
  instance_count         = 2
  ami                    = data.aws_ami.amzlinux.id
  instance_type          = "t2.micro"
  key_name               = "terraform-key"
  monitoring             = true
  vpc_security_group_ids = ["sg-08b25c5a5bf489ffa"] # Get Default VPC Security Group ID and replace
  subnet_id              = "subnet-4ee95470"        # Get one public subnet id from default vpc and replace
  user_data              = file("apache-install.sh")

  tags = {
    Terraform   = "true"
    Environment = "dev"
  }
}
```

## Block-9: **Moved Blocks**
```
moved {
  from = "aws_instance.my_ec2"
  to   = "aws_instance.my_new_ec2"
}
```

## Block-10: **Import Blocks**
```
import {
  to = aws_vpc.my_vpc
  id = "vpc-0b37c8c1dd6d9c791"
}

resource "aws_vpc" "my_vpc" {}
```
