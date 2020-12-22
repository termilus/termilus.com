---
title: Reverse SSH Server Deployment and Configuration
date: 2020-12-22T12:26:28.196Z
published: true
tags:
  - Reverse SSH
  - IoT connectivity
  - AutoSSH
  - port forwarding
cover_image: ../../static/images/uploads/logo.png
description: >-
  This guide explains how to deploy a Reverse SSH Server in AWS. The Reverse SSH
  Server allows multiple clients to connect to the server via SSH on a
  persistent basis. The clients are configured to port forward their own local
  SSH service to allow administration of all clients simply by connecting to the
  Reverse SSH Server and then SSH-ing to the appropriate local ports.


  This appliance allows clients behind NAT'd environments with private IP addresses to connect outbound to the Reverse SSH Server and be administrate-able. The server and its clients are configured to auto-reconnect if either one gets rebooted.
---
### **What**

The Reverse SSH Server is an appliance which provides persistent connections to other servers or IoT devices even when those endpoints are deployed in private IP environments and even if the endpoints reboot unexpectedly.

This persistent connection is provided by SSH over port 22 by default. Authentication to the server is specific to each client and key-based authentication is used. Old clients can be removed from the server and new clients can be added at anytime.

### **Why**

The Reverse SSH Server eliminates the need to track and remember multiple, dozens, or hundreds of IP addresses of endpoint systems which need to be managed. All clients auto-connect to the server and maintain the reverse connection at all times. The reverse SSH server meets the following needs:

1. Persistent client connections to single server for easier administration.
2. Custom client names for ease of administration.
3. Connection durability - auto reconnects even across reboots.
4. Ability to change server listening port on initial setup from 22 to 443 or any other port.


### **How**

