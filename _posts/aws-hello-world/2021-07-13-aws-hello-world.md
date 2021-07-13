---
layout: post
title: AWS Hello World Webserver with Terraform
date: 2021-07-13
tags: [aws, terraform, cloud]
---

There are many tutorials online that show you how to spin up an EC2 instance on AWS using Terraform and SSH into the instance, but most of these tutorials assume that you will be using a default VPC or a VPC that has already been created. This tutorial will start from scratch, showing every AWS service needed along the way, and eventually spin up an EC2 instance that will display “Hello, World!” in your browser. You will also be able to SSM into the instance after it is deployed.

The steps that will be covered are as follows:

-   Setup
-   Authenticate with AWS
-   Create your VPC
-   Create your Subnet
-   Create an Internet Gateway for your VPC
-   Create a Route Table for your VPC
-   Associate the Route Table with your Subnet
-   Create a Security Group for your VPC
-   Create an IAM Instance Profile for your VPC
-   Create an EC2 instance and put it in your Subnet
-   Deploy Resources
-   SSM into EC2

Let’s get started!

## Setup

Before we do anything else, we need to set up the directory we’ll be working in. Create a folder named “aws-tutorial” and add a file named “main.tf.” Usually you would separate code into different files based on what it does, but to keep things simple, we will add all of the code to the main.tf file.

You also want to make sure that you have [Terraform downloaded](https://www.terraform.io/downloads.html) before moving on to the next step.

## Authenticate with AWS

There are several ways to authenticate with AWS while using Terraform. In this tutorial we will be using static credentials, but if you want to explore other options, you can look at the [Terraform Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs).

To authenticate and specify versions, add the following two blocks to your main.tf file:

```terraform
provider "aws" {
  region = "us-east-1"
  access_key = “my-access-key”
  secret_key = “my-secret-key”
}

terraform {
  required_version = ">= 0.13"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.37.0"
    }
  }
}
```

You can get your access key and secret key in the AWS console. Also, be sure that the region you add is the same as the region you're using in your AWS account. For more information about this check out the [AWS Documentation](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html)

## Create your VPC

Creating a VPC with Terraform can be done by simply adding the following code block to your main.tf file:

```terraform
resource "aws_vpc" "tutorial-vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
}

```

The cidr_block parameter assigns 65,536 IP addresses to our VPC, and the enable_dns_hostnames enables the VPC to use DNS hostnames. For more information about these parameters and to see a list of other optional parameters, you can check out the [Terraform Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc).

## Create your Subnet

Creating a Subnet with Terraform is also a simple task, and it can be done by adding the following code block to your main.tf file:

```terraform
resource "aws_subnet" "tutorial-subnet" {
  vpc_id                  = aws_vpc.tutorial-vpc.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "us-east-1a"
}
```

The vpc_id references the VPC that we created in the last step, and the cidr_block assigns 256 IP addresses to our Subnet. We set map_public_ip_on_launch to true so that we have a public IP for the EC2 that we will create later in the tutorial. For more information about these parameters and to see a list of other optional parameters, you can check out the [Terraform Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/subnet).

## Create an Internet Gateway for your VPC

Internet Gateways allow your VPC to reach the internet and the internet to reach your VPC. This is necessary because we will need to be able to connect to our VPC from the internet in order to see our Hello World program running in the browser. Add the following code block to your main.tf file to create an Internet Gateway:

```terraform
resource "aws_internet_gateway" "tutorial-igw" {
  vpc_id = aws_vpc.tutorial-vpc.id
}
```

Here we just reference our VPC. For more information about creating Internet Gateways with Terraform, you can check out the [Terraform Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/internet_gateway).

## Create a Route Table for your VPC

A Route Table is used to configure a set of rules that determine where network traffic in a subnet is directed. We create a Route Table by adding the following code block to our main.tf file:

```terraform
resource "aws_route_table" "tutorial-rt" {
  vpc_id = aws_vpc.tutorial-vpc.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.tutorial-igw.id
  }
}
```

We first reference our VPC, and then we define a route for the Internet Gateway that we just created. For more information about these parameters and to see a list of other optional parameters, you can check out the [Terraform Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route_table).

## Associate the Route Table with your Subnet

To associate the route table that your just made with your Subnet, add the following code block to your main.tf file:

```terraform
resource "aws_route_table_association" "tutorial-rta" {
  subnet_id      = aws_subnet.tutorial-subnet.id
  route_table_id = aws_route_table.tutorial-rt.id
}
```

Here we just reference the ID of our Subnet and our Route Table. For more information about this resource, check out the [Terraform Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/main_route_table_association).

## Create a Security Group for your VPC

Security Groups are similar to firewalls. They are stateful, meaning that if traffic is allowed from one direction, it is implicitly allowed in the other direction. By default, AWS allows all outbound traffic with Security Groups. We add a Security Group to our VPC by adding the following code block to our main.tf file:

```terraform
resource "aws_security_group" "tutorial-sg" {
  vpc_id = aws_vpc.tutorial-vpc.id
  egress {
    from_port   = 443
    protocol    = "tcp"
    to_port     = 443
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    description = "HTTP from VPC"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

Egress is for outbound traffic, and ingress is for inbound traffic. We open port 443 (https) outbound so that we can SSM into the instance, and we open port 80 (http) inbound so that we can access our Hello World program running in the browser. For more information about Security Groups, you can check out the [Terraform Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group).

## Create an IAM Instance Profile for your VPC

There are four code blocks that are needed to add the IAM Instance Profile to your VPC:

```terraform
resource "aws_iam_instance_profile" "default_ssm_instance_profile" {
  name = "DefaultSSMProfile"
  role = aws_iam_role.default_ssm_role.name
}

resource "aws_iam_role" "default_ssm_role" {
  name               = "DefaultSSMProfileRole"
  path               = "/"
  assume_role_policy = <<EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": "sts:AssumeRole",
            "Principal": {
               "Service": "EC2.amazonaws.com"
            },
            "Effect": "Allow",
            "Sid": ""
        }
    ]
}
EOF
}

data "aws_iam_policy" "AmazonSSMManagedInstanceCore" {
  arn = "arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore"
}

resource "aws_iam_role_policy_attachment" "default_ssm_policy_attachment" {
  policy_arn = data.aws_iam_policy.AmazonSSMManagedInstanceCore.arn
  role       = aws_iam_role.default_ssm_role.name
}
```

The only reason we need to do this is so that we can SSM into the instance after it has been deployed. Here’s a list of the documentation for each resource that we used:

-   [aws_iam_instance_profile](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_instance_profile)
-   [aws_iam_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role)
-   [aws_iam_role_policy_attachment](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role_policy_attachment)

## Create an EC2 instance and put it in your Subnet

The last code block that we will be adding to main.tf is for our EC2 instance:

```terraform
resource "aws_instance" "tutorial-EC2" {
  ami                  = "ami-0dc2d3e4c0f9ebd18"
  subnet_id            = aws_subnet.tutorial-subnet.id
  instance_type        = "t2.micro"
  iam_instance_profile = aws_iam_instance_profile.default_ssm_instance_profile.name
  vpc_security_group_ids = [aws_security_group.tutorial-sg.id]
  user_data         = <<-EOF
                #! /bin/bash
                sudo yum update
                sudo yum install -y httpd
                sudo systemctl start httpd
                sudo systemctl enable httpd
                echo "
Hello, World!

" | sudo tee /var/www/html/index.html
        EOF
}
```

This code block will also start an Apache web server and create an index.html file that says “Hello, World!” The AMI must be an AMI that has SSM installed on it, so here we use the Amazon Linux 2 AMI. If you are also using the us-east-1 region, this AMI should work for you, but if you are using another region, you will have to find out the ID of the Amazon Linux 2 AMI by using the AWS console. Another parameter that’s important here is the iam_instance_profile parameter. This is necessary for us to be able to SSM into the instance. For more information about these parameters and to see a list of other optional parameters, you can check out the [Terraform Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance).

## Deploy Resources

To deploy the resources, open your terminal and cd into your aws-tutorial directory. The first command that you’ll use is `terraform init`.

```bash
$ terraform init
> ...
```

This command will configure your directory.

The next command that you’ll use is `terraform plan`.

```bash
$ terraform plan
> ...
```

This command will show you what resources will be deployed. It will also show you if you have any syntactic errors in your code.

Next, you’ll run the `terraform apply` command.

```bash
$ terraform apply
> ...
```

This command will deploy your resources into AWS. After running this command you will be able to see the resources in your AWS console.

Now you can get your public IP for your EC2 from the AWS console and type it in the browser: `http:<public-ip>:80`. You should now see your “Hello, World!” message.
Finally, you can run `terraform destroy` to undeploy all of the resources.

```bash
$ terraform destroy
> ...
```

## SSM into EC2

Now, for fun, we will SSM into the instance. You can do this by going to EC2 in the AWS console and clicking “connect” at the top of the screen. Select the Session Manager tab and click “connect.” After connecting to the instance you can go to the index.html file where “Hello, World!” is written and change it to something else. You do this by typing `sudo vi /var/www/html/index.html` and updating the file. Now, you should be able to see your changes when you go to `http:<public-ip>:80`.

## Conclusion

Thank you for taking the time to read this tutorial! I hope it was helpful, and if you have any suggestions for this tutorial or future content, be sure to send me an email at jimmyeneville@gmail.com
