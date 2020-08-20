---
title: "Add External Domain AWS"
date: 2020-08-19T21:21:09-07:00
tags:
  - aws
  - route53
  - domain name
draft: false
---

I have a domain name, `norell.dev` that is regeisterd outside of AWS. I would like to use it for my development within AWS, but Amazon doesn't support the `.dev` domain name. Google is the gTLD owner, and Amazon and Google don't play well together. Even though Amazon won't let me transfer my domain to be registered in Route53, I can still configure it to be used using Hosted Zones.

I will follow [this guide](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/migrate-dns-domain-inactive.html) to get it set up.

# Steps

1. Log into AWS Route53
2. Add Hosted Zone
3. Log into registrar (Namecheap, GoDaddy, Google, etc.)
4. Change domain specific name server to the ones provided by AWS Route53
5. Wait 24-48 hours if you had DNSSEC enabled

# Configure website to use domain

Now that the domain is in Route53, I can set the domain for use by the website hosted in S3. You can find more information on that from the [previous post](../host-website-in-aws).

## Get Hosted Zone into Terraform State

Becaues I created the hosted zone manually, I need to let Terraform know it exists, so that it doesn't try to create it again. The first step to import the state is add the resource. In `domain.tf`, I added the following

```tf
resource "aws_route53_zone" "norell_dev" {
  name = "norell.dev"
}
```

The matches the name of my resource and zone that is already in Route53. I now need to grab the AWS ID of the zone. This can be found on the [Hosted zones panel](https://console.aws.amazon.com/route53/v2/hostedzones#), under the table entry **Hosted zone ID**. In my case, mine is `Z0892995169ZQ16UVKJTE`

I then ran the terraform import command to transfer the current state to tfstate

```sh
terraform import aws_route53_zone.norell_dev Z0892995169ZQ16UVKJTE
```

## Create the alias

Once the state was imported to terraform, I was able to add the alias route of `www.norell.dev` to the endpoint of the S3 bucket hosting the website.

Adding to `domain.tf`

```tf
resource "aws_route53_record" "norell_dev" {
  zone_id = aws_route53_zone.norell_dev.zone_id
  name    = "www.norell.dev"
  type    = "A"

  alias {
    name                   = aws_s3_bucket.website.website_endpoint
    zone_id                = aws_s3_bucket.website.hosted_zone_id
    evaluate_target_health = true
  }
}
```

This grabs the endpoint and hosted zone ID from the bucket, allowing for no hardcoded values for the alias.
