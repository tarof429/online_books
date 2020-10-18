# Part 4: S3 Example

This example shows how to create an S3 bucket. It will create a bucket called `my-bucket` and upload one file to it with public read access. Also note that the bucket has two tags!

{% code title="main.tf" %}
```bash
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
    }
  }
}

resource "aws_s3_bucket" "mybucket" {
  bucket = "my-bucket"
  versioning {
    enabled = false
  }
  tags = {
    Name        = "taro-special-bucket"
    Environment = "Prod"
  }
  acl    = "private"
}

resource "aws_s3_bucket_object" "upload_hello" {
  bucket = aws_s3_bucket.mybucket.id
  key = "hello.txt"
  source = "hello.txt"
  etag   = filemd5("hello.txt")
  acl = "public-read"
}

# Configure the AWS Provider
provider "aws" {
  region = "us-west-2"
}
```
{% endcode %}