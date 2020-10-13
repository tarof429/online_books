# Part 3: Advanced Terraform

This section is based on the Youtube video at `https://www.youtube.com/watch?v=SLB_c_ayRMo&t=2608s`. See the end of this section for the full reference.

## Commands

1. *terraform state list*

```bash
$ terraform state list
aws_eip.one
aws_instance.web-server-instance
aws_internet_gateway.gw
aws_network_interface.eth0
aws_route_table.prod-route-table
aws_route_table_association.subnet-1-prod-route-table-assocation
aws_security_group.allow_web
aws_subnet.subnet-1
aws_vpc.prod-vpc
```

2. *terraform state show*

```bash
$ terraform state show aws_eip.one
# aws_eip.one:
resource "aws_eip" "one" {
    associate_with_private_ip = "10.0.1.100"
    association_id            = "eipassoc-0aa9e081ce5b34d9d"
    domain                    = "vpc"
    id                        = "eipalloc-017035a5a171342a7"
    instance                  = "i-0d6e8629edd026e78"
    network_interface         = "eni-0ba66e91aadabbed2"
    private_dns               = "ip-10-0-1-100.us-west-2.compute.internal"
    private_ip                = "10.0.1.100"
    public_dns                = "ec2-44-239-71-96.us-west-2.compute.amazonaws.com"
    public_ip                 = "44.239.71.96"
    public_ipv4_pool          = "amazon"
    vpc                       = true
}
```

## Tricks

We can print the public IP of our EC2 intance by adding an output section and running *terraform apply*.

{% code title="main.tf" %}
```bash
...
output "server_public_ip" {
  value = aws_eip.one.public_ip
}
output "server_state" {
  value = aws_instance.web-server-instance.instance_state
}
...
```
{% endcode %}

Result: 

```bash
$ terraform apply
Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

server_public_ip = 44.239.71.96
server_state = running
```

Alternatively, use *terraform output*.

```bash
$ terraform output
server_public_ip = 44.239.71.96
server_state = running
```

If you're uncertain whether the values are up-to-date, run *terraform refresh*. This may be better than doing an apply that would change live infrastructure.

## Deleting a resource using terraform

We can use terraform to delete a single resource without deleting the entire infrastructure.

```bash
$ terraform state list
aws_eip.one
aws_instance.web-server-instance
aws_internet_gateway.gw
aws_network_interface.eth0
aws_route_table.prod-route-table
aws_route_table_association.subnet-1-prod-route-table-assocation
aws_security_group.allow_web
aws_subnet.subnet-1
aws_vpc.prod-vpc
$ terraform destroy -target aws_instance.web-server-instance
```

## Applying a single resource

We can do the converse and apply a single resource.

```bash
 terraform apply -target aws_instance.web-server-instance
 ```

## Variables

We can use variables instead of hard-coded values.

```bash
variable "aws_region" {
  description = "Region where we deploy the instance"
  #default = "us-west-2"
  #type = "string"
}

# Configure the AWS Provider
provider "aws" {
  #region = "us-west-2"
  region = var.aws_region
}
```

When we run `terraform plan`, terraform will prompt the user for a value.

To avoid prompts, we can specify the var value.

```bash
terraform apply -var "aws_region=us-west-2" --auto-approve
```

An alternative way is to create the file `terraform.tfvars` and define the variables there.

{% code title="terraform.tfvars" %}
```bash
aws_region = "us-west-2"
```
{% endcode %}

If the variable file is not called *terraform.tfvars*, we need to specify the path to the file explicitly.

```bash
terraform apply -var-file foo.tfvars
```

You can also assign default values to variables. These will be used if the user doesn't set the value.

You can also set the type of the variable. For example, *string*. 

## References

<a href="https://www.youtube.com/watch?v=SLB_c_ayRMo&t=2608s">Terraform Course - Automate your AWS cloud infrastructure by freeCodeCamp.org</a>
