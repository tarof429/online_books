# Part 1: Installation and Introduction

This section discusses how to install terraform on ArchLinux and perform basic operations to create infrastructure. The content is based on a Youtube video at `https://www.youtube.com/watch?v=SLB_c_ayRMo&t=2608s`. See the end of this section for the full reference.

## The docker provider

1. The official instructions for installing Terraform are at https://learn.hashicorp.com/tutorials/terraform/install-cli. For ArchLinux, install the terraform package.

    ```bash
    sudo pacman -S terraform
    ```

2. Verify the installation.

    ```bash
    terraform -help
    ```

3. Per the instructions on the terraform website, enable tab completion.

    {% code title="main.tf" %}
    ```bash
    terraform -install-autocomplete
    ```
    {% encode %}

4. Create a directory called terraform-docker-demo

    ```bash
    mkdir terraform-docker-demo
    ```

5. cd to it and add the following content to a file called `main.tf`.

    {% code title="main.tf" %}
    ```bash
    terraform {
    required_providers {
        docker = {
        source = "terraform-providers/docker"
        }
    }
    }

    provider "docker" {}

    resource "docker_image" "nginx" {
    name         = "nginx:latest"
    keep_locally = false
    }

    resource "docker_container" "nginx" {
    image = docker_image.nginx.latest
    name  = "tutorial"
    ports {
        internal = 80
        external = 8000
    }
    }
    ```
    {% endcode %}

6. . Initialize the project.

    ```bash
    terraform init
    ```

7. Provision the NGINX container. Answer Yes at the command prompt.

    ```bash
    terraform apply
    ```

8. Confirm that the container has been created.

    ```bash
    [docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
    8088bfe6e7c6        992e3b7be046        "/docker-entrypoint.…"   3 seconds ago       Up 2 seconds        0.0.0.0:8000->80/tcp   tutorial
    ```

9. Visit http://localhost:8000

10. Stop the container

    ```bash
    terraform destroy
    ```

## The AWS provider

1. Create a new project folder called `aws-demo`

    ```bash
    mkdir aws-demo
    ```

    {% hint style="info" %}
    To be able to use the aws provider, first install the AWS CLI first and run `aws configure` so that your credentials are set. You can create an AWS user just for API access and the AWS resources you want to use.
    {% endhint %}

2. Create `main.tf` with the following content.

    {% code title="main.tf" %}
    ```bash
    terraform {
        required_providers {
            aws = {
            source  = "hashicorp/aws"
            }
        }
    }
    ```
    {% endcode %}

3. Configure the AWS Provider

    {% code title="main.tf" %}
    ```
    provider "aws" {
    region = "us-west-2"
    }
    ```
    {% endcode %}


4. Run `terraform init`

    ```bash
    terraform init
    ```

5. Run `aws apply`

    ```bash
    terraform apply
    ```

    {% hint style="info" %}
    To apply a plan and skip the prompts, run `terraform apply --auto-approve`.
    {% endhint %}

6. Confirm the new instance in the AWS console at `https://us-west-2.console.aws.amazon.com/ec2`

7. Tear down the instance.

    ```bash
    terraform destroy
    ```

Now the interesting thing about Terraform is that destroying a resource is as simple as commenting out the resource and running `terraform apply` again! This makes it easy to track changes to infrastructure as we never have to explicitly destroy any resourcs. Terraform is declarative, so that every resource in a plan file will be matched with your cloud infrastructure. If anything doesn't match, Terraform will go through the steps to ensure that the state matches. 

### Creating a VPC

The basic syntax for creating a VPC can be seen at `https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc`. A basic code snippet is shown below.

{% code title="main.tf" %}
```bash
resource "aws_vpc" "first_vpc" {
  cidr_block = "10.0.0.0/16"

    tags = {
      Name = "first_vpc"
  }
}
```
{% endcode %}

### Creating a subnet

The basic syntax for creating a subnet can be set at `https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/subnet`. A basic code snippet is shown below.

{% code title="main.tf" %}
```bash
resource "aws_subnet" "main" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"

  tags = {
    Name = "Main"
  }
}
```
{% endcode %}

To create a subnet for `first_vpc`, what we want to do is the following:

{% code title="main.tf" %}
```bash
resource "aws_subnet" "subnet-1" {
  vpc_id     = aws_vpc.first_vpc.id
  cidr_block = "10.0.1.0/24"

  tags = {
    Name = "prod-subnet"
  }
}
```
{% endcode %}

What's key here is that the vpc_id references the ID of `first_vpc`. We can easily do this by copying `"aws_vpc" "first_vpc"`, remove the quotes, and adding `id` at the end! 

{% hint style="info" %}
The order in which resources are defined do no matter in Terraform. For example, it is possible to declare subnets before VPCs.
{% endhint %}

### Files and directories created by Terraform

- `.terraform`: this directory will be created when running `terraform init`. If you remove this directory, terraform will complain about missing plugins.

- `terraform.tfstat` Every time you run `terraform apply`, this file will be modified. This file must not be removed because it contains state information.

## References

<a href="https://www.youtube.com/watch?v=SLB_c_ayRMo&t=2608s">Terraform Course - Automate your AWS cloud infrastructure by freeCodeCamp.org</a>
