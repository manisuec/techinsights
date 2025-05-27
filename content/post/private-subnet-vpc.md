---
layout: post
title: "Deploy EC2 in Private Subnet & Securely Enable Internet Communication"
description: Secure critical infrastructure in AWS by deploying EC2 instances in a private subnet. Prevent direct internet access and protect databases & application servers
date: 2023-01-23 00:00:00 +0530
lastmod: 2025-05-27T00:00:30
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1674130623/private_vpc_nat_ufnejw.png']
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1675429691/AWS-Dark_jwzawt.jpg'
tags: ['mongodb', 'aws']
keywords: 'mongodb,aws,vpc,private subnet,nat,internet,private,secure,communication'
categories: ['Mongodb']
url: 'mongodb/db-ec2-private-subnet-vpc'
---

Enterprise-grade security architectures demand that critical infrastructure like databases, application servers etc must be deployed securely in AWS and deploying ec2 instances in private subnet is a must. Instances launched in a private subnet have only private ip address (No public IP) and hence these instances are completely isolated from direct internet access. This architectural pattern creates a robust security perimeter that effectively eliminates the attack surface for external threats while maintaining the flexibility to establish controlled outbound connectivity when required.

## Deployment diagram {#fig_1}

![private subnet](https://res.cloudinary.com/dkiurfsjm/image/upload/v1674130623/private_vpc_nat_ufnejw.png) 

## Understanding NAT Gateway Functionality

A network address translation (NAT) gateway allows EC2 instances to establish outbound connections to resources on internet without allowing inbound connections to the EC2 instance. It's not possible to use the private IP addresses assigned to instances in a private VPC subnet over the internet. Instead, you must use NAT to map the private IP addresses to a public address for requests. Then, you must map the public IP address back to the private IP addresses for responses.

## Why Deploy EC2 instances in Private Subnets?

Why do ec2 instances deployed on private subnets then need to communicate with internet? Few scenarios could be like 

- download software 
- download latest patches of software for security purposes
- send emails / sms to your users

## Set up NAT gateway

The deployment will look similar to [figure 1](#fig_1).
Follow the steps mentioned below:

* [Create a public VPC subnet](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html#Add_IGW_Create_Subnet) specifically designated to host your NAT gateway. This subnet must be properly sized to accommodate the NAT gateway and any other internet-facing components such as bastion hosts or application load balancers.

* [Create and attach an internet gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html#Add_IGW_Attach_Gateway) to your VPC. The internet gateway serves as the connection point between your VPC and the internet, enabling bidirectional communication for resources in public subnets.

* [Create a custom route table](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html#Add_IGW_Routing) for your public subnet that includes a default route (0.0.0.0/0) pointing to the internet gateway. This routing configuration enables internet connectivity for resources deployed in the public subnet.

* Verify that the network access control list (ACL) for your public VPC subnet allows inbound traffic from the private VPC subnet; refer [Work with network ACLs](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html#Add_IGW_Routing).
* [Create a public NAT gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html#Add_IGW_Routing) within the public subnet. Select the appropriate bandwidth allocation based on your expected traffic patterns, keeping in mind that NAT gateways can scale automatically but incur charges based on data processing volume.

* [Create](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html#using-instance-addressing-eips-allocating) and [associate](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html#using-instance-addressing-eips-associating) your new or existing Elastic IP address that can be whitelisted in external systems and ensures consistency in outbound IP addressing.

* [Update the route table](https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateway-scenarios.html#public-nat-gateway-routing) of your private VPC subnet to point internet traffic to your NAT gateway. This configuration ensures that all outbound internet traffic from private subnet instances is routed through the NAT gateway.

* [Test your NAT gateway](https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateway-scenarios.html#public-nat-gateway-testing) by pinging the internet from an instance in your private VPC subnet.

### Security Considerations and Best Practices

**Defense in Depth Strategy**

While NAT gateways provide network-level isolation, implement additional security controls including:

- **Security Groups**: Configure restrictive security group rules that allow only necessary ports and protocols
- **Network ACLs**: Implement subnet-level access controls as an additional security layer
- **VPC Flow Logs**: Enable comprehensive network traffic logging for security monitoring and compliance
- **AWS CloudTrail**: Monitor API calls and configuration changes to networking components

**Cost Optimization**

NAT gateways incur charges based on hourly usage and data processing volume. Consider implementing VPC endpoints for AWS services to reduce NAT gateway data processing costs, and evaluate whether NAT instances might be more cost-effective for lower-volume workloads.

**High Availability Architecture**

Deploy NAT gateways in multiple Availability Zones to ensure business continuity. Configure route tables in each private subnet to use the NAT gateway within the same Availability Zone to minimize cross-AZ data transfer charges.

## Conclusion

I hope this post helps you in seting up your environment for deploying your applications in a VPC with public and private subnets. And helps you understand on how the ec2 instances in private subnet can connect to internet.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.

## Conclusion

Implementing EC2 instances within private subnets with controlled internet access through NAT gateways represents a fundamental security architecture pattern that balances protection requirements with operational flexibility. This approach provides organizations with the ability to maintain strict network isolation for critical infrastructure while enabling necessary outbound connectivity for system maintenance, third-party integrations, and user communications.

The architectural pattern described in this guide forms the foundation for enterprise-grade AWS deployments and should be considered a mandatory component of any production workload that handles sensitive data or requires regulatory compliance. I hope this post helps you in seting up your environment for deploying your applications in a VPC with public and private subnets. And helps you understand on how the ec2 instances in private subnet can connect to internet.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.

