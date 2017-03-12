# aws-infrastructure

## Introduction

**So, you want to get a server up and running in Amazon Web Services (AWS)?**

I pulled this project together after talking to people who heard amazing things about AWS but struggled to get a test server up and running with internet connectivity.

Some of the most hardcore indie developers and technical educators I know have pulled their hair out trying to get a server deployed. There's no shame in that! They're smart people, but they're super-busy with their day jobs and just need a helping hand to get started.

## What does this project give you?

### The short version

[infrastructure.cloudformation.yaml](./infrastructure.cloudformation.yaml) is an AWS CloudFormation template which will set up everything you need to be able to deploy a server.

I'll also provide a quick example for launching a sample Apache server, but anything more than that is beyond the scope of this project.

For live, customer-facing services, you'll want to research Auto-Scaling Groups, Elastic Load Balancers, and other strategies for making your service secure and highly available. I might cover this in the future, but for now I encourage you to experiment and learn.

Don't worry about "breaking" your AWS account by experimenting. I'll show you how to easily drop and redeploy all your infrastructure so you can start again from a clean slate.

### The long version

#### Overview

[infrastructure.cloudformation.yaml](./infrastructure.cloudformation.yaml) is an AWS CloudFormation template which will provide:

* A Virtual Private Cloud (VPC)
* An Internet Gateway
* A NAT Gateway
* 4 subnets
  * 2 "private" subnets (across 2 availability zones)
  * 2 "public" subnets (across 2 availability zones)
* Appropriate IP traffic routing between the subnets, NAT gateway and Internet Gateway.
  * The "private" subnets will have outgoing internet connectivity (via the NAT Gateway to Internet Gateway), but will not be reachable from the internet
  * The "public" subnet will have both incoming and outgoing internet connectivity (via the Internet Gateway)

#### Why "public" and "private" subnets?

The **"public"** subnets are fairly self-explanatory. Any EC2 instances deployed in there will have public IP addresses assigned to them, and they'll have both incoming and outgoing internet connectivity.

This might sound like the only kind of subnet you want, but consider: do you *really* want to fully expose your EC2 instances to the wild internet?

Ideally, you want to consider deploying your services to EC2 instances in the **"private"** subnets. These instances will be able to make outgoing internet connections (e.g. to download updates and other software) but since they won't have public IP addresses, they won't be reachable from the internet.

So, how can EC2 instances in the "private" subnets host services if they're not reachable? You'd deploy an *Elastic Load Balancer (ELB)*. The ELB sits on the public internet, receives the traffic for your service, and balances requests across all the instances of your service in the "private" subnets. This topic is a bit too big to go into here, and I'd love to go into it another day.

Suffice to say, I strongly urge you to do your own investigation into making your services secure and highly available.

## Usage

### Demo

Here's a screencast I recorded, where I:
1. Download [infrastructure.cloudformation.yaml](./infrastructure.cloudformation.yaml) from GitHub.
1. Create an S3 bucket and upload the template.
1. Use CloudFormation to deploy the template.
1. Launch an EC2 instance in the new infrastructure.
1. Deploy a test web server onto the new EC2 instance, and show that it works.
1. Tear down the web server.
1. Tear down all the new infrastructure.

[![Demonstration](http://img.youtube.com/vi/T1WUev4-YKM/0.jpg)](http://www.youtube.com/watch?v=T1WUev4-YKM)

### Deploying the infrastructure

1. [Create an AWS account](https://aws.amazon.com/) and log in.
1. Open the S3 service.
1. Create an S3 bucket and upload [infrastructure.cloudformation.yaml](./infrastructure.cloudformation.yaml) into it.
1. Copy the public URL of the uploaded file. It should look something like `https://s3-<region>.amazonaws.com/<bucket>/infrastructure.cloudformation.yaml`.
1. Open the CloudFormation service.
1. At the top-right of the page, select the geographical region to deploy into. If you don't have any special legal or locational requirements then anywhere will do.
1. Select "Create Stack".
1. Select "Specify an Amazon S3 template URL" and paste in the public URL of the template.
1. Select "Next".
1. Enter a simple name to identify this set of infrastructure. I just call mine "infrastructure".
1. Select any two different availability zones.
1. Select "Next".
1. Leave the default options, and select "Next".
1. On the review page, select "Create" to start creating the resources.
1. You should see a stack appear with the status "CREATE_IN_PROGRESS". *It will take several minutes to complete.*
  * If the creation ends with a failure message, raise an issue and I'll try to help.
  * You will need to manually delete the failed stack. Select it, then select "Actions / Delete Stack".
  * If the deletion fails, you will need to contact AWS support. I cannot provide support on your account.
1. When the stack's status becomes "CREATE_COMPLETE" then you're all done!

### Deploying a test web server

1. Open the EC2 service.
1. Launch a new instance.
1. Select the default Amazon Linux AMI.
1. Select the default instance type (which should be `t2.micro`).
1. When prompted for network options:
  1. When asked to pick a network, make sure you select the item named `vpc-<YOUR STACK NAME>`.
  1. When asked to pick a subnet, make sure you select one named "public".
1. Select the default storage options.
1. When prompted for tags, set the "Name" tag to "web-demo".
1. When prompted for a security group:
  1. SSH (on port 22) should be included already. Under "Source", change "Custom" (or "Anywhere") to "My IP".
  1. Add another rule to allow HTTP (port 80).
1. On the review page, select "Launch".
1. When prompted to select a key pair:
  1. Select "Create a new key pair".
  1. Name it "demo".
  1. Select "Download Key Pair".
  1. Select "Launch Instance".
1. Return to the EC2 service. You should see your new instance in the "running" state. It is booting up, and will be available in a few minutes.
1. To connect to the EC2 instance:
  * **Linux/macOS:**
    1. Open a terminal and navigate to the downloaded key.
    1. Run: `chmod 400 demo.pem`
    1. Run: `ssh -i demo.pem ec2-user@<IP>` (replace `<IP>` with the public IP address of the instance)
    1. When warned about the authenticity of the host:
      1. Double-check you really did enter the correct IP address.]
      1. Select "yes".
  * **Windows:**
    1. Apologies for this bit being a bit vague. I don't have Windows.
    1. Download [PuTTYgen](http://www.chiark.greenend.org.uk/~sgtatham/putty/) and use it to convert `demo.pem` to `demo.ppk`.
    1. Download [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/) and use it to create an SSH connection to your EC2 instance. You'll need to configure PuTTY to authenticate with `demo.ppk` and connect to your instance's public IP address.
1. When you're connected:
  1. Run: `sudo yum update --assumeyes`
  1. Run: `sudo yum install httpd24 --assumeyes`
  1. Run: `sudo service httpd start`
    * Ignore any warnings you see, as long as the final line says `[ OK ]`.
1. Back on your own machine, open a web browser at `http://<IP>`. You should see an Apache test page.

Congratulations! You just deployed a web server to AWS!

### Shutting down the web server

1. Log into your AWS account.
1. Open the EC2 service.
1. Select your EC2 instance.
1. Select "Actions / Instance State / Terminate".
1. Select "Yes, Terminate" to confirm.
1. Open the list of Security Groups, and delete the Security Group you created above.

### Deleting all this infrastructure from AWS

This will delete all the resources created by the CloudFormation template above. Follow these steps if you want to permanently destroy all the resources, or just remove them so you can re-deploy to a clean slate later.

1. Log into your AWS account.
2. Open the CloudFormation service.
3. Select the stack.
4. Select "Actions / Delete Stack".

The deletion could take a long time (even longer than the creation time), and could fail if it ends up taking too long. I urge you to contact AWS support if you have problems deleting the stack.

## Issues

### CloudFormation can't delete the VPC!

This most likely means you left something behind in the VPC. Are you *sure* you deleted all the Security Groups you created?

## Very important note about security

**While I use this infrastructure myself, I present it without any guarantee of security.**

Vulnerabilities in internet-connected systems are being constantly discovered, and AWS isn't magically protected from ne'er-do-wells.

Even if you had a "perfect" technologically-secured environment, your AWS account is still at risk of social-engineering attacks.

I strongly urge you to do your own security research.

## Very important note about costs (i.e. real life money)

When you sign up for an AWS account, you have to provide billing details. This is because **the resources you create will accrue costs, and AWS will expect you to pay.**

Some resources cost only cents per day, but others cost more. Some resources are always free, and some are free for a limited amount of time.

Costs also vary between regions.

It's impossible for me to detail the kinds of costs you could expect in AWS because I don't know what region you'll be deploying into, or if AWS will tweak their costs right after I publish this. I also can't guarantee that you won't mistakenly launch a high-cost resource.

At the time of writing this, the resources provided by this project cost fractional cents and are basically free. However, I strongly urge you to follow AWS guidelines on setting up billing alerts and monitoring your costs. I absolutely am not responsible for paying your bills.

## …but setting those warnings aside…

AWS is affordable, fun, and generally a safe place to learn about cloud computing. It's currently the only cloud provider I recommend to friends and colleagues.

Don't get stuck just reading about it. Dive in, try it out, learn from your mistakes, and have fun!
