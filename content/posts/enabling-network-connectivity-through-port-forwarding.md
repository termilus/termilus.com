---
title: Enabling AWS Network Connectivity Through Port Forwarding
date: 2024-04-09T19:17:51.701Z
published: true
tags:
  - Case Study
  - Multi-Port Forwarding
  - AWS
  - Better Together
cover_image: ../../static/images/uploads/mpfsaws.png
description: T﻿his blog post is a case study that captures the Multi-Port
  Forward Server value proposition. It outlines traffic routing challenges faced
  by many organizations and highlights examples of successful Multi-Port Forward
  Server deployments.
---
## What is the Multi-Port Forward Server?

Establishing network connectivity to systems and services hosted in AWS can be a challenging and expensive endeavor - but it doesn't have to be! The Termilus Multi-Port Forward Server is a cost-effective solution that enables organizations to expose numerous internal network services to the Internet using only a single elastic IP address. By implementing the Multi-Port Forward Server, an organization can securely connect to all its AWS services on custom-defined ports, even when those services are running on the same ports, internally.

## What problems does the Multi-Port Forward Server solve?

When many systems are deployed into an AWS VPC, those systems need to be administered. However, administration faces the following challenges:

* Attaching elastic IP addresses (EIPs) to all of those systems is expensive and increases the organization’s internet-facing attack surface.
* Security groups are attached to all these instances creating configuration sprawl for network security administrators.
* The ports on which these services are listening may not be accessible by customer organizations who may only allow outbound internet connections on common ports like 80 or 443.

## How does the Multi-Port Forward Server solve these issues?

The Multi-Port Forward Server acts as a transparent gateway for all inbound traffic destined for your target systems. The appliance only uses one EIP and can be configured to listen on one or more ports. When traffic is sent to one of its listening ports, it conducts port address translation (PAT) in addition to network address translation (NAT) to reroute the traffic to the appropriate host inside the VPC. This flexibility can even be leveraged to reroute traffic to systems outside of the VPC.

## What are some example configurations of the Multi-Port Forward Server?

Once deployed, the Multi-Port Forward Server is quickly and easily configured. Some configuration examples for our clients include:

* Listening on TCP port 25565 and UDP port 19132, routing all traffic back to a residential IP address to host a Minecraft server. This masked the client’s home IP address from others connecting to the Minecraft server.
* Publicly listening on TCP port 443, routing all traffic to a Redshift database listening on port 5439. This allowed the client to bypass outbound port restrictions to administer their Redshift cluster.
* Publicly listening on TCP ports 2222, 2223, 2224, 2225, etc. routing inbound traffic to different internal SSH servers all listing locally on port 22. This allowed our client to drastically reduce their EIP count by leveraging a singular Multi-Port Forward Server appliance to reach all of their internal systems.