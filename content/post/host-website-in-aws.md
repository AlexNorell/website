---
title: "Host Website in AWS"
date: 2020-08-18T23:06:28-07:00
tags:
  - terraform
  - AWS
draft: true
---

I'm want to deploy a copy of this website to AWS S3, and set up all of the necessary infrastructure using terraform. This post will go over the steps that were taken to achieve this.

# Set up provider

This will be using AWS, so the terraform provider must be set. This will use the latest terraform version, `0.13`.

The provider is defined below, and located in `main.tf`

```tf
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "3.2.0"
    }
  }
}
```

# Set up backend

This project will use S3 as the backend for storing the terraform state files. I created an S3 bucket using the AWS cli.

```sh
aws s3 mb s3://alexnorell-tfstates --region us-west-1
```

I then added the following to `backend.tf`

```tf
terraform {
  backend "s3" {
    bucket = "alexnorell-tfstates"
    key    = "website"
    region = "us-west-1"
  }
}
```

