# aws-infrastructure

## Introduction

[infrastructure.cloudformation.yaml](./infrastructure.cloudformation.yaml) is an AWS CloudFormation template which will provide:

* A Virtual Private Cloud (VPC)
* An Internet Gateway
* Two subnets (across two availability zones)
* Subnet routing: EC2 instances in the public subnets will have full Internet connectivity

If you want to deploy private subnets in additional to all of the above then follow up with [private-subnets.cloudformation.yaml](./private-subnets.cloudformation.yaml) to get:

  * Two NAT gateways (across two availability zones)
  * Two private subnets (across two availability zones)
  * Subnet routing: EC2 instances in the private subnets will have outgoing Internet connectivity, but will not be reachable from the Internet (unless you put them behind a bastion, Elastic Load Balancer, or some other router)

## Security

While I use this infrastructure myself, **I present it without any guarantee of security**.

Vulnerabilities in Internet-connected systems are being constantly discovered, and AWS isn't magically protected from ne'er-do-wells.

Even if you had a "perfect" technologically-secured environment, your AWS account is still at risk of social-engineering attacks.

I strongly urge you to do your own security research.

## Costs (i.e. real life money)

When you sign up for an AWS account, you have to provide billing details. This is because **the resources you create will accrue costs, and AWS will expect you to pay.**

Some resources cost only cents per day, but others cost more. Some resources are always free, and some are free for a limited amount of time. Costs also vary between regions.

It's impossible for me to detail the kinds of costs you could expect in AWS because I don't know what region you'll be deploying into, or if AWS will tweak their costs right after I publish this. I also can't guarantee that you won't mistakenly launch a high-cost resource.

I strongly urge you to follow AWS guidelines on setting up billing alerts and monitoring your costs. **I absolutely am not responsible for paying your bills.**

## License

This work is licensed under the MIT License. There are no restrictions on how you may reuse it, but I'd appreciate some credit if you find the templates helpful.
