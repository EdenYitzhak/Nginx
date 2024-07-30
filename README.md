# DevOps Home Assignment - Detailed Process Documentation

## Overview
The objective of this assignment was to deploy an NGINX instance that is publicly accessible and displays the text "yo this is nginx" upon access. The task involved using Terraform for infrastructure setup, Docker for containerization, and AWS for deployment. Below is a step-by-step explanation of the process, including the challenges faced and how they were addressed.

## Step-by-Step Process

### Step 1: AWS Infrastructure Setup
**Tools Used**: Terraform (Infrastructure as Code - IaC)

Process:
1. Initialize Terraform configuration.
2. Define AWS resources (EC2 instances, VPCs, Security Groups).
3. Apply the Terraform plan to provision the infrastructure.

### Step 2: Docker Containerization
**Tools Used**: Docker

Process: dedicated branch

### Step 3+4: Public Access
**Tools Used**: AWS, NGINX

Process:
1. Configure the NGINX server to serve the application.
2. Set up public access by configuring the AWS Security Groups and network settings.

### GitHub Workflow for Deployment
**Tools Used**: GitHub Actions

Process:
1. Define the CI/CD pipeline using GitHub Actions.
2. Configure the workflow to automate the deployment process.

## Code Snippets

### Terraform Script- to initiate EC2 instance
#### main.tf
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main_vpc" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public_subnet" {
  vpc_id                  = aws_vpc.main_vpc.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
}

resource "aws_subnet" "private_subnet" {
  vpc_id     = aws_vpc.main_vpc.id
  cidr_block = "10.0.2.0/24"
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main_vpc.id
}

resource "aws_eip" "nat" {
  vpc = true
}

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public_subnet.id
}

resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}

resource "aws_route_table_association" "public_association" {
  subnet_id      = aws_subnet.public_subnet.id
  route_table_id = aws_route_table.public_rt.id
}

resource "aws_route_table" "private_rt" {
  vpc_id = aws_vpc.main_vpc.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat.id
  }
}

resource "aws_route_table_association" "private_association" {
  subnet_id      = aws_subnet.private_subnet.id
  route_table_id = aws_route_table.private_rt.id
}

resource "aws_security_group" "nginx_sg" {
  vpc_id = aws_vpc.main_vpc.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "nginx_instance" {
  ami                    = "ami-0427090fd1714168b"
  instance_type          = "t2.micro"
  subnet_id              = aws_subnet.private_subnet.id
  vpc_security_group_ids = [aws_security_group.nginx_sg.id]
  key_name               = "my-key-pair"

  user_data = <<-EOF
              #!/bin/bash
              yum update -y
              amazon-linux-extras install docker -y
              service docker start
              usermod -a -G docker ec2-user
              docker run -d -p 80:80 nginx:latest /bin/sh -c "echo 'yo this is nginx' > /usr/share/nginx/html/index.html; nginx -g 'daemon off;'"
              EOF

  tags = {
    Name = "NginxInstance"
  }
}
