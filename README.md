
## Step-by-Step Process

### Step 1: AWS Infrastructure Setup
**Tools Used**: Terraform (Infrastructure as Code - IaC)

1. Initialize Terraform configuration.
2. Using AWS resources (EC2 instances, VPCs, Security Groups).
3. Apply the Terraform plan to provision the infrastructure.


## using AWS CLI
aws configure
Access key ID :******R6GGD
Secret access key: ******jJOhQ87
region name (us-east-1)
output format (json)

## EC2 Instance
given my Private IP 10.0.2.139 public IP:3.231.241.165
-> terraform init
-> terraform apply

Instance: i-0f0f27870be0928a6
![image](https://github.com/user-attachments/assets/a3087360-bb1c-486e-8af2-c76f34a48954)
![image](https://github.com/user-attachments/assets/5e700edf-5f82-43db-88c5-383df2626b3c)
![image](https://github.com/user-attachments/assets/ddd3c432-d9c4-4d52-bc04-627a57b66bd6)


![image](https://github.com/user-attachments/assets/84781680-bb4e-4dc2-b666-2302fa753b12)

Configured a Virtual Private Cloud (VPC) with public and private subnets.
Created security groups to allow HTTP (port 80) and SSH (port 22) access.


### Terraform Script- to initiate EC2 instance
#### main.tf-for step 1-AWS
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

```

## Challenges:

Internet Connectivity Issues: The instance could not access the internet, it seems that when Im sshing to the the host all

you can see here:

```hcl
C:\Users\97250>ssh -i C:\Users\97250\Terraform\devops-assignment\my-key-pair.pem ec2-user@3.231.241.165
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
Last login: Tue Jul 30 16:05:40 2024 from 147.235.215.142
```

![i-0f0f27870be0928a6](https://github.com/user-attachments/assets/f2d29fad-8c78-43f3-adf0-433c2e7b6151)

but encountered an issue with yum update due to network connectivity problems, which was the only thing missing to integrate the running instance to docker
and since I'm unable to use yum update to install Docker directly on the EC2 instance due to network issues,
I Verified Docker functionality locally using Docker Desktop.

which you will see on the step 2 of Docker Containerization which you will eventually execute and get a web with "yo this is nginx" 
