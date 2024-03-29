---
title: AWS and Azure Multi-Port Forward Setup and Configuration
date: 2024-03-04T22:30:33.149Z
published: true
tags:
  - AWS
  - Multi-Port Forward
  - Azure
  - NAT
  - PAT
cover_image: ../../static/images/uploads/multi-portforwardserverlogo.png
description: This guide explains how to deploy and configure a Multi-Port
  Forward Server in AWS and Azure. The Multi-Port Forward Server is designed to
  listen on multiple user-defined ports and redirect inbound TCP or UDP traffic
  to user-defined destination hosts and ports. This creates a versatile network
  address translation (NAT) and port address translation (PAT) appliance in your
  AWS VPC or Azure subscription.
---
### **Estimated Deployment Time**

1﻿0 Minutes

### **What**

Port forwarding or port address translation (PAT) is a method of changing the destination port of network traffic by using a forward proxy.

Network address translation (NAT) is the process of redirecting inbound network traffic destined for an exposed public IP address, to internal private non-routable IP addresses. The process is then reversed when the internal IPs communicate with external IPs.

### **Why**

The Multi-Port Forward Server serves both purposes of NAT-ing at PAT-ing when multiple streams of traffic need to be redirected. This is especially useful in the following scenarios:

1. Numerous internal servers listen on the same port, but must all be internet-exposed on a single IP address
2. Only one IP address can be whitelisted but multiple external hosts need to be accessed
3. Access to internal systems is needed externally, and traffic must traverse on port 443 or 80 to bypass a network firewall dropping all other ports

### **How**

For this tutorial we will deploy a single Multi-Port Forward Server that redirects inbound traffic to multiple hosts internally and externally.

Let’s assume the following network architecture:

* A VPC or subscription configured with a 172.31.0.0/16 CIDR range
* Two subnets configured:

  * One private, with no internet access (172.31.1.0/24)
  * One public, with an internet gateway configured for internet access (172.31.0.0/24)

  ![Subnets](../../static/images/uploads/portforward-subnets.png)
* A Redshift cluster deployed to the private subnet which will be listening on its default TCP port, 5439
* An Ubuntu server deployed to the private subnet listening for inbound SSH connections on TCP port 22
* A Port Forward Server appliance deployed to the public subnet will be listening on TCP ports 443, 80, 53, and 22

  * Inbound TCP traffic on port 443 will be proxied to the Redshift server in the private subnet, destined for port 5439
  * Inbound TCP traffic on port 80 will be proxied to the Ubuntu server in the private subnet, destined for port 22
  * Inbound TCP traffic on port 53 will be proxied to portquiz.net, an external host listening on port 3389

  ![Deployed Multi-Port Forward Server](../../static/images/uploads/deployedmultiportforwardserver.png)
* The VPC security group or subscription NSG needs to be configured to allow:

  * Inbound ports 443, 80, 53, and 22 from an authorized IP address
  * Inbound ports 5439 and 22 from the public subnet into the private subnet

    ![VPC Security Group](../../static/images/uploads/multivpcsecuritygroup.png)

Here is an architectural diagram of the setup we’ve just constructed:

![Multi-Port Forward Diagram](../../static/images/uploads/multi-portforwarddiagram.png)

Now that the infrastructure is setup, we're ready to deploy the Multi-Port Forward Server into the public subnet.

### How to deploy the Multi-Port Forward Server:

1. Deploy the [AWS Multi-Port Forward Server](https://aws.amazon.com/marketplace/pp/prodview-of6usifaxnkcw) or [Multi-Port Forward Server for Azure](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/fatalsecurity1604924013537.multi-port-forward-server?tab=Overview) appliance from the marketplace into your public subnet
2. If in AWS, disable source and destination checks on the instance, [more info here](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html#EIP_Disable_SrcDestCheck)
3. SSH into the Multi-Port Forward Server
4. Open the /etc/multiportforward/​multiportforward.config file with vim or nano and update the SPORT, DHOST, DPORT and PROTOCOL entries (source port, destination host, destination port and connection protocol respectively) in JSON format; key names should be descriptive and should not contain any spaces
5. Save the multiportforward.config file
6. Reboot the server

![Multi-Port Forwarding Working](../../static/images/uploads/multi-portforwardserversetup.png)

And that’s it! Your Multi-Port Forward Server should now be forwarding:

* All incoming port 443 TCP traffic is being directed to your Redshift cluster, deployed in your private subnet, listening on port 5439
* All incoming port 80 TCP traffic is being directed to your Ubuntu SSH server, deployed in your private subnet, listening on port 22
* All incoming port 53 TCP traffic is being directed to an external RDP server, listening on port 3389

You can test the connection to your Redshift cluster by entering the public DNS of your Multi-Port Forward Server as the target host using psql:

`psql -h ec2-12-34-56-78.compute-1.amazonaws.com -U awsuser -d dev -p 443`

Again, the same host name can be used to verify the connection to your internal Ubuntu SSH host:

`ssh -i ~/.ssh/Ubuntu.pem ubuntu@ec2-12-23-34-78.compute-1.amazonaws.com -p 80`

And similarly, use the same hostname when connecting via remote desktop.

### Video:

[![Multi-Port Forward Server Walk-Through](../../static/images/uploads/multiportforwardserverstill.png)](https://youtu.be/teJK8vap4xQ "Multi-Port Forward Server Appliance Walk-Through")

### Troubleshooting

* Traffic not being forwarded?

  * Check the security groups to ensure traffic is being permitted inbound on expected ports.
  * Check ACLs to ensure traffic is being permitted outbound as expected.
  * Ensure source and destination checks have been disabled on the instance: [more info here](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html#EIP_Disable_SrcDestCheck)

### FAQ:

* Does the Multi-Port Forward Server support all regions? Yes, all regions are supported.
* S﻿hould I use the account root user to deploy this appliance? No, we recommend using a non-root user to deploy this appliance.
* I﻿s encrypted traffic decrypted by the appliance at any point, either in-transit or at rest? No, the Multi-Port Forward Server facilitates traffic to and from hosts but never decrypts encrypted traffic.
* D﻿oes the Multi-Port Forward Server require any billable services? Yes, in addition to the appliance subscription, the Multi-Port Forward Server will incur the cost of the EC2 instance selected. For example, the licensing cost for a t2.micro instance would be $0.02 for the software and $0.012 for the EC2 instance, yielding a total cost of $0.032 per hour.
* H﻿ow large of an EC2 instance should we deploy? We recommend a t3.medium for most production use cases.
* How can I check the health of my Multi-Port Forward Server? Check the health of your server by viewing its status in the "Instance State" column in the EC2 Dashboard.
* How can I install system or software patches? Once the system is rebooted, it will auto-install all requisite updates and upgrades.
* How do I handle a non-responsive Multi-Port Forward Server? This is very rare, but a reboot should resolve the issue.
* Is there technical assistance available to help troubleshoot? Yes, we're available to assist and will typically respond within 1 business day. We can be reached by emailing: support at termilus.com