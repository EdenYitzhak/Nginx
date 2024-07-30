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

Process:
1. Write a Dockerfile to containerize the NGINX application.
2. Build the Docker image.
3. Run the Docker container locally to test.

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

### Terraform Script
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "nginx" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "nginx-instance"
  }
}

