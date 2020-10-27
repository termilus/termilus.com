---
title: AWS Port Forward Setup and Config
date: 2020-10-26T11:37:57.504Z
published: true
tags:
  - AWS
  - Port Forward
cover_image: ../../static/images/uploads/portforwardserverlogo.png
description: This guide explains how to deploy and configure a Port Forward
  Server in AWS. The deployed server redirects inbound traffic to either
  internal or external hosts using custom source and destination ports.
---
### **What**

Port forwarding or port address translation (PAT) is a method of altering the destination port of traffic by using a forward proxy.

### **Why**

There are multiple possible reasons to use a PAT server for port forwarding. Here are just a few:

1. Running multiple NAT’d servers with identical services competing for one port.
2. Changing the default destination port to a more common port to bypass layer 4 firewall restrictions.
3. Configuring round robin DNS where multiple AWS port forward servers can redirect traffic to one application server.

### **How**

For this tutorial we will use Redshift deployed to a private subnet in AWS as our example.

In this scenario, we are on-prem, and outbound traffic to port 5439 is blocked by the network firewall.

To get around this restriction, we will configure our Port Forward Server to publicly listen on port 443, and forward the incoming traffic to Redshift on port 5439. All incoming traffic will be IP whitelisted to only allowing traffic from our local IP address.

Let’s assume the following network architecture:

* A VPC configured with a 172.31.0.0/16 CIDR range.
* Two subnets configured:
* One private, with no internet access (172.31.1.0/24)
* One public, with an internet gateway configured for internet access (172.31.0.0/24)

    

  ![Subnets](../../static/images/uploads/portforward-subnets.png)
* A Redshift cluster deployed to the private subnet which will be listening on its default port, 5439.
* A Multi-Port Forward Server appliance deployed to the public subnet will be listening on port 443. Incoming traffic will be proxied to the Redshift server in the private subnet, destined for port 5439.

    

  ![Deployed Port Forward Server](../../static/images/uploads/deployedportforwardserver.png)
* The VPC security group needs to be configured to allow:

  * Inbound ports 22 and 443 from an authorized IP address
  * Inbound port 5439 from the public subnet into the private subnet
  * And inbound traffic from the security group itself allowing internal VPC traffic

    ![VPC Security Group](../../static/images/uploads/vpcsecuritygroup.png)

Here is an architectural diagram of the setup we’ve just constructed:

![Port Forward Diagram](../../static/images/uploads/portforwarddiagram.png)

Once the networking is configured properly, we can deploy and configure the Multi-Port Forward Server appliance.

### Steps to deploy the Multi-Port Forward Server are as follows:

1. Deploy the AWS [Multi-Port Forward Server](https://aws.amazon.com/marketplace/pp/Fatal-Security-Multi-Port-Forward-Server/B086QWQQXX) (or alternatively, a single AWS [Port Forward Server](https://aws.amazon.com/marketplace/pp/B07TDD4LM6)) appliance from the marketplace into your public subnet
2. SSH into the Multi-Port Forward Server
3. Edit the JSON entries to reflect your desired port forwarding configuration; multiple entries can be added to forward multiple ports to multiple servers
4. Enter a custom key name for your entry (sshbastion, rdpbastion, etc.); this is can be set to anything
5. Update the corresponding source ports (SPORT), destination ports (DPORT), and the destination hosts (DHOST)
6. Save the portforward.config file
7. Reboot the server
8. Ensure that the AWS Security Group permits inbound traffic from the source IPs to the respective source ports

![Port Forwarding Working](../../static/images/uploads/loggedinconfiguredloggedout.png)

And that’s it! Your Multi-Port Forward Server should now be forwarding all incoming port 443 traffic to your Redshift cluster, deployed on your private subnet, listening on port 5439.

You can test this by connecting to your Redshift cluster by entering the public DNS of your Multi-Port Forward Server as the target host using psql:

`psql -h ec2-3-89-128-232.compute-1.amazonaws.com -U testuser -d dev -p 443`