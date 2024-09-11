---
layout: post
title: "Deploy EC2 in Private Subnet & Securely Enable Internet Communication"
description: Secure critical infrastructure in AWS by deploying EC2 instances in a private subnet. Prevent direct internet access and protect databases & application servers
date: 2023-01-23 00:00:00 +0530
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1674130623/private_vpc_nat_ufnejw.png']
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1675429691/AWS-Dark_jwzawt.jpg'
tags: ['mongodb', 'aws']
keywords: 'mongodb,aws,vpc,private subnet,nat,internet,private,secure,communication'
categories: ['Mongodb']
url: 'mongodb/db-ec2-private-subnet-vpc'
---

Critical infrastructure like databases, application servers etc must be deployed securely in AWS and deploying ec2 instances in private subnet is a must. Instances launched in a private subnet have only private ip address (No public IP) and hence these instances can never be reached from internet. This prevents direct attacks on the system and applications in the system.

## Deployment diagram {#fig_1}

![private subnet](https://res.cloudinary.com/dkiurfsjm/image/upload/v1674130623/private_vpc_nat_ufnejw.png) 

## NAT Gateway: Short description

A network address translation (NAT) gateway allows EC2 instances to establish outbound connections to resources on internet without allowing inbound connections to the EC2 instance. It's not possible to use the private IP addresses assigned to instances in a private VPC subnet over the internet. Instead, you must use NAT to map the private IP addresses to a public address for requests. Then, you must map the public IP address back to the private IP addresses for responses.

## Why?

Why do ec2 instances deployed on private subnets then need to communicate with internet? Few scenarios could be like 

- download software 
- download latest patches of software for security purposes
- send emails / sms to your users

## Set up NAT gateway

The deployment will look similar to [figure 1](#fig_1).
Follow the steps mentioned below:

* [Create a public VPC subnet](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html#Add_IGW_Create_Subnet) to host your NAT gateway.
* [Create and attach an internet gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html#Add_IGW_Attach_Gateway) to your VPC.
* [Create a custom route table](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html#Add_IGW_Routing) for your public subnet with a route to the internet gateway.
* Verify that the network access control list (ACL) for your public VPC subnet allows inbound traffic from the private VPC subnet; refer [Work with network ACLs](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html#Add_IGW_Routing).
* [Create a public NAT gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html#Add_IGW_Routing) 
* [Create](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html#using-instance-addressing-eips-allocating) and [associate](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html#using-instance-addressing-eips-associating) your new or existing Elastic IP address.
* [Update the route table](https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateway-scenarios.html#public-nat-gateway-routing) of your private VPC subnet to point internet traffic to your NAT gateway.
* [Test your NAT gateway](https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateway-scenarios.html#public-nat-gateway-testing) by pinging the internet from an instance in your private VPC subnet.

## Conclusion

I hope this post helps you in seting up your environment for deploying your applications in a VPC with public and private subnets. And helps you understand on how the ec2 instances in private subnet can connect to internet.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.

