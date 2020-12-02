---
title: Azure Multi-Port Forward Setup and Config
date: 2020-12-01T12:54:54.119Z
published: true
tags:
  - Azure
  - Multi-Port Forward
cover_image: ../../static/images/uploads/multi-portforwardserverlogo.png
description: This guide explains how to deploy and configure a Multi-Port
  Forward Server in Azure. The Multi-Port Forward Server is designed to listen
  on multiple user-defined ports and redirect inbound traffic to user-defined
  hosts (internal or external) and ports. This creates a versatile network
  address translation (NAT) and port address translation (PAT) appliance in your
  Azure VPC.
---
### **What**

Port forwarding or port address translation (PAT) is a method of changing the destination port of network traffic by using a forward proxy.

Network address translation (NAT) is the process of redirecting inbound network traffic destined for an exposed public IP address, to internal private non-routable IP addresses. The process is then reversed when the internal IPs communicate with external IPs.

### **Why**

The Azure Multi-Port Forward Server serves both purposes of NAT-ing at PAT-ing when multiple streams of traffic need to be redirected. This is especially useful in the following scenarios:

1. Numerous internal servers listen on the same port, but must all be internet-exposed on a single IP address
2. Only one IP address can be whitelisted but multiple external hosts need to be accessed
3. Access to internal systems is needed externally, and traffic must traverse on port 443 or 80 to bypass a network firewall dropping all other ports

### **How**

For this tutorial we will deploy a single Multi-Port Forward Server that redirects inbound traffic to multiple hosts internally and externally.

Let’s assume the following network architecture:

* A virtual network (vnet) configured with a 10.1.0.0/16 CIDR range
* Two subnets configured:

  * One private, with no internet access (10.1.1.0/24)
  * One public, with an internet gateway configured for internet access (10.1.0.0/24)

  ![Subnets](../../static/images/uploads/multi-portforwardsubnets.png)

* A Windows system deployed to the private subnet which will be listening for RDP connections on its default port, 3389
* An Ubuntu server deployed to the private subnet listening for inbound SSH connections on port 22
* A Port Forward Server appliance deployed to the public subnet will be listening on ports 443, 80, 53, and 22

  * Inbound traffic on port 443 will be proxied to the Windows system in the private subnet, destined for port 3389
  * Inbound traffic on port 80 will be proxied to the Ubuntu server in the private subnet, destined for port 22
  * Inbound traffic on port 53 will be proxied to portquiz.net, an external host listening on port 3389

  ![Deployed Multi-Port Forward Server](../../static/images/uploads/multi-portforwardserverdeployed.png)
* The network security groups needs to be configured to allow:

  * Inbound ports 443, 80, 53, and 22 from an authorized IP address
  * Inbound ports 3389 and 22 from the public subnet into the private subnet

    ![VPC Security Group](../../static/images/uploads/multivpcsecuritygroup.png)

Here is an architectural diagram of the setup we’ve just constructed:

![Multi-Port Forward Diagram](../../static/images/uploads/multi-portforwarddiagram.png)

Now that the infrastructure is setup, we're ready to deploy the Multi-Port Forward Server into the public subnet.

### How to deploy the Multi-Port Forward Server:

1. Deploy the AWS [Multi-Port Forward Server](https://aws.amazon.com/marketplace/pp/B086QWQQXX/?ref=_ptnr_termilus_ampfsc) appliance from the marketplace into your public subnet
2. SSH into the Multi-Port Forward Server
3. Open the portforward.config file and update the SPORT, DHOST and DPORT entries (source port, destination host, and destination port respectively) in JSON format; key names should be descriptive
4. Save the portforward.config file
5. Reboot the server

![Multi-Port Forwarding Working](../../static/images/uploads/validatedmultiportforward.png)

And that’s it! Your Multi-Port Forward Server should now be forwarding:

* All incoming port 443 traffic is being directed to your Redshift cluster, deployed in your private subnet, listening on port 5439
* All incoming port 80 traffic is being directed to your Ubuntu SSH server, deployed in your private subnet, listening on port 22
* All incoming port 53 traffic is being directed to an external RDP server, listening on port 3389

You can test the connection to your Redshift cluster by entering the public DNS of your Multi-Port Forward Server as the target host using psql:

`psql -h ec2-12-34-56-78.compute-1.amazonaws.com -U awsuser -d dev -p 443`

Again, the same host name can be used to verify the connection to your internal Ubuntu SSH host:

`ssh -i ~/.ssh/Ubuntu.pem ubuntu@ec2-12-23-34-78.compute-1.amazonaws.com -p 80`

And similarly, use the same hostname when connecting via remote desktop.