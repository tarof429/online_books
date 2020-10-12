# Part 2: Sample Project

This section is based on the Youtube video at `https://www.youtube.com/watch?v=SLB_c_ayRMo&t=2608s`. See the end of this section for the full reference.

## Goals

1. Create a VPC
2. Create Internet Gateway
3. Create Custom Route Table
4. Create a Subnet
5. Assocate subnet with the route table
6. Create a security group to allow ports 22, 80, and 443
7. Create a network interface with an IP in the subnet that was created in step 4
8. Assign an elastic IP to the network interface created in step 7
9. Create an Ubuntu server and install/enable apache2

We will create a new directory called `sample-webapp` and do all our terraform work there.

## The code

{% code title="main.tf" %}
```bash
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
    }
  }
}

# Configure the AWS Provider
provider "aws" {
  region = "us-west-2"
}

# 1. Create a VPC
resource "aws_vpc" "prod-vpc" {
  cidr_block = "10.0.0.0/16"
  tags = {
      Name = "Production"
  }
}

# 2. Create Internet Gateway
resource "aws_internet_gateway" "gw" {
  vpc_id = aws_vpc.prod-vpc.id
}

# 3. Create Custom Route Table
resource "aws_route_table" "prod-route-table" {
  vpc_id = aws_vpc.prod-vpc.id

  # Allow all traffic out to the Internet
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.gw.id
  }

  route {
    ipv6_cidr_block        = "::/0"
    gateway_id = aws_internet_gateway.gw.id
  }

  tags = {
    Name = "Prod"
  }
}

# 4. Create a Subnet
resource "aws_subnet" "subnet-1" {
  vpc_id     = aws_vpc.prod-vpc.id
  cidr_block = "10.0.1.0/24"
  availability_zone = "us-west-2a"

  tags = {
    Name = "prod-subnet"
  }
}

# 5. Assocate subnet with the route table
resource "aws_route_table_association" "subnet-1-prod-route-table-assocation" {
  subnet_id      = aws_subnet.subnet-1.id
  route_table_id = aws_route_table.prod-route-table.id
}

# 6. Create a security group to allow ports 22, 80, and 443
resource "aws_security_group" "allow_web" {
  name        = "allow_web_traffic"
  description = "Allow web inbound traffic"
  vpc_id      = aws_vpc.prod-vpc.id

  ingress {
    description = "HTTPS"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }


  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "allow_web"
  }
}

# 7. Create a network interface with an IP in the subnet that was created in step 4
resource "aws_network_interface" "eth0" {
  subnet_id       = aws_subnet.subnet-1.id
  private_ips     = ["10.0.1.100"]
  security_groups = [aws_security_group.allow_web.id]
}

# 8. Assign an elastic IP to the network interface created in step 7
resource "aws_eip" "one" {
  vpc                       = true
  network_interface         = aws_network_interface.eth0.id
  associate_with_private_ip = "10.0.1.100"
  depends_on                = [aws_internet_gateway.gw]
}


# 9. Create an Ubuntu server and install/enable apache2
resource "aws_instance" "web-server-instance" {
  ami           = "ami-06e54d05255faf8f6"
  availability_zone = "us-west-2a"
  instance_type = "t2.micro"
  key_name = "freecodecamp"
  
  network_interface {
    network_interface_id = aws_network_interface.eth0.id
    device_index         = 0
  }

  user_data = <<-EOF
    #!/bin/bash
    sudo apt update -y
    sudo apt install apache2 -y
    sudo systemctl start apache2
    sudo mkdir -p /var/www/html
    sudo bash -c 'echo my first web server > /var/www/html/index.html'
  EOF

  tags = {
    Name = "my-ec2-instance"
  }
}
```
{% endcode %}

Once in the instance, you can visit `http://169.254.169.254/latest/user-data` to see and validate the user data.

## References

<a href="https://www.youtube.com/watch?v=SLB_c_ayRMo&t=2608s">Terraform Course - Automate your AWS cloud infrastructure by freeCodeCamp.org</a>
