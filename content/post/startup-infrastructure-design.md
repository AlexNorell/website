---
title: "Startup Infrastructure Design"
date: 2021-05-29T09:57:26-07:00
draft: true
---

A company asked me to design their infrastructure as part of an interview. It is a pretty transparent attempt to get free work, but hey, this actually makes a good post and is a fun exercise.

## Prompt

### Motivation

As we scale, we need to streamline our code deployment and infrastructure to be as efficient and robust as possible so that we can iterate quickly. These are some of the challenges that you’d be solving.

### Problem

1. What tools/processes do you use for cloud infrastructure management, security, and auditing?
2. We are a small, fast moving startup. With this in mind, how would you think about build vs buy?
3. How would you construct a CI/CD pipeline on AWS with a [React frontend](https://reactjs.org/), [NodeJS backend](https://nodejs.org/en/), and pool of worker jobs?
4. How would you setup an [ElasticSearch](https://www.elastic.co/elasticsearch/) cluster for search with 100 million records in it, mostly free text across multiple languages?

## Tools

Let's break this down into a couple different sections:

1. Code Management and Process
2. Infrastructure
3. Security & Auditing

### Code Management and Process

All of your code needs to be in version control system, preferably `git`. It should be pushed to a hosted system like [GitHub](https://github.com), [GitLab](https://gitlabs.com), [BitBucket](https://bitbucket.com), or [CodeCommit](https://aws.amazon.com/codecommit/). You should be using a [GitHub Flow](https://guides.github.com/introduction/flow/) style of development so that you can iterate quickly on your code. This system focuses on short lived code branches with frequent merges of code into a main branch. Releases are done from this main branch. Releases are defined as [Git Tags](https://git-scm.com/book/en/v2/Git-Basics-Tagging) which can then trigger the continuous deployment system to start a deployment. 

All code needs to be reviewed by someone else before it gets merged into the main branch. The culture of the team needs to be built around reviewing the code constructively and efficiently. Changes to the code base need to be small, with a general rule of thumb that you shouldn't be changing more that 100 lines in any given pull request. Doing meaningful code review needs to be prioritized higher than starting new features. Technical Debt is very hard to get rid of and having a system of thoughtful review helps keep that at bay.

Depending on the size of the team and the amount of work that needs to be done, there are a couple of different ways to plan work. You will want some way to track deficiencies in the code and new product features. Most of the hosted git systems have builtin issue tracking and project management products that are free or included with your per-user cost. For technical teams, these make the most sense because they can be integrated directly into the code review process. 

If you want something that is a bit more general purpose, there are a ton of options out there. [Jira](https://www.atlassian.com/software/jira) is the industry incumbent, but I really wouldn't recommend using it. There are a ton of different SASS products out there for doing project management and they all offer free tiers/trials. Trying out a couple and finding which ever one makes the most sense for your workflow makes a lot of sense. 

### Infrastructure

For managing your infrastructure, it really depends on what you envision for your product in the future. Do you have any plans of moving off of AWS to another Cloud provider? Do you need to host fully separated instances of your product for other organizations, either internal to their infrastructure or cloud? Let's just assume that we're going to stay on AWS for right now.

For defining and configuring all of the infrastructure and services, there are a couple of different options in AWS. You could do everything manually if there really wasn't a large footprint of services. This would be the cheapest upfront cost in terms of time and engineering resources, but you wouldn't be able to replicate or change this easily. Another option would be to use [AWS CloudFormation](https://aws.amazon.com/cloudformation/), which is a YAML-based configuration tool for configuring all aspects of AWS's services. These configuration are committed into a repository and deployed automatically.

Another option is [Terraform](https://www.terraform.io/),  which allows for a unified way to configure many different services across all of the cloud providers. It also supports configuration of applications that might be deployed into instances instead of just managed services.

The benefit of using CloudFormation over Terraform is that there is usually Day 1 support for new products directly in CloudFormation. Terraform is probably a better choice if you ever plan to move to a different cloud provider or want to go multi-platform. Terraform has larger learning curve, but in my opinion is a better product for infrastructure definition.

If you want to spend a lot of time and build a custom solution, you can interface with all of the cloud providers using just their APIs. I've done it for some specific application deployments, but it really is overkill. You can also use something like [Terraform CDK](https://github.com/hashicorp/terraform-cdk), which is powerful but also pretty overkill for React + Node application.

You need some way to metric your application and collect application logs. There a lot of services out there to accomplish this, but it makes the most sense to just use AWS CloudWatch. This way you don't need to ship your logs out to these services and pay for the AWS egress traffic. You can set up alarms to alert if particular metrics are breached. 

### Security & Auditing

At a bare minimum, you need to have dependency scanning set up for your node application to alert you if any of your dependencies have vulnerabilities. This is a very easy and free thing to do with GitHub's [Dependabot](https://dependabot.com/). This will even open up pull requests to update your dependencies automatically. You will also want to set up some sort of static security analysis tool like [SonarCloud](https://sonarcloud.io/) or [ShiftLeft](https://www.shiftleft.io/). These scan through the code itself and surface any security issues they identify. These are also both managed services meaning that there is no infrastructure to manage and maintain.

You will also want to set up some sort of active scanning of your infrastructure using a tool like [Tenable](https://www.tenable.com/) to check your APIs and public facing infrastructure. This will alert you if anything is deployed that wasn't caught by any of the other checks. It will also detect if anything new is opened up that you didn't expect, like a known command and control port.

For auditing, you can use [AWS Audit Manager](https://aws.amazon.com/audit-manager/), which audits your AWS usage and helps keep your AWS usage within compliance of a particular standard. Auditing is also a lot easier if everything, from application design to development process, is documented before you start the process. These processes should also meet whatever the requirements of a particular certification requires.

## Build vs Buy

There is no question on which one you should do, **buy**. Building and deploying a web application is a solved problem. There is nothing you are doing that sounds incredibly custom, and instead you should be focusing your engineering efforts on building out your product, rather than custom solutions.

From what it sounds like, the company has a high margin product, with a low amount of concurrent users. Building and deploying a web application using managed serverless solutions that can automatically scale when demand increases in the future sounds like the best approach.

## CI/CD Pipeline

A Continuous Deployment pipeline requires something to deploy and something to deploy into. Let's start with identifying the components:

- React Frontend
- User authentication
- NodeJS backend components
- ElasticSearch database

### Application Design

![Architecture Design](/images/react_app/architecture.svg)

The React Frontend can be deployed directly to [Amazon CloudFront](https://aws.amazon.com/cloudfront/) so that we can leverage AWS's fast content delivery network. CloudFront plus the DNS settings configured in Route53 should all be configured statically 

## ElasticSearch Cluster

