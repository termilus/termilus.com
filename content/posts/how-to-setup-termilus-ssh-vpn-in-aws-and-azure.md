---
title: How to Setup Termilus SSH VPN in AWS and Azure
date: 2021-05-20T20:22:25.688Z
published: false
tags:
  - SSH VPN
  - Remote Access
description: Setup a client for your SSH VPN and see how to get connected.
---
## What

The Termilus SSH VPN server is a remote access solution similar to a traditional VPN, however traffic traverses an SSH tunnel as opposed to PPTP, L2TP, IPSEC etc.

Currently, the Termilus SSH VPN has been tested and confirmed to be working on:

* Ubuntu 20.04
*

## Why

* The Termilus SSH VPN provides remote access to your AWS VPC or Azure subscription. This means once you've connected you'll be able to access internal systems to your subscription or VPC on private subnets.
* It provides a secure, encrypted tunnel to your subscription or VPC providing protection from unscrupulous internet service providers or Wi-Fi hotspots.
* It masks your actual origin IP address. Meaning, any sites you visit or systems you connect to will see the Termilus SSH VPN's public IP address as the connection's origin.

## How

1. Deploy the Termilus SSH VPN into your Azure subscription or AWS VPC. The server will need to be assigned a publicly accessible IP address.
2. Connect to the Termilus SSH VPN: `ssh -i ~/.ssh/Termilus.pem ec2-user@ec2-54-91-77-201.compute-1.amazonaws.com`
3. Run the sshvpn command to create a new SSH VPN user: `sudo sshvpn -a ubuntu`
4. Exit the SSH session with the server: `exit`
5. Use SFTP to retrieve the client installer script: `sftp -i ~/.ssh/Termilus.pem ec2-user@ec2-54-91-77-201.compute-1.amazonaws.com:/home/ec2-user/SSHVPN-ubuntu-Client.sh ./SSHVPN-ubuntu-Client.sh`
6. Make the client installer script executable: `chmod +x SSHVPN-ubuntu-Client.sh`
7. Run the installer script: `sudo ./SSHVPN-ubuntu-Client.sh`
8. The SSH VPN can now be toggled on with:

   Linux: `sudo service sshvpn-ubuntu start`

   MacOS: `sudo launchctl load /Library/LaunchDaemons/org.termilus.sshvpn.plist`

   And can be toggled off again with:

   Linux: `sudo service sshvpn-ubuntu stop`

   MacOS: `sudo launchctl unload /Library/LaunchDaemons/org.termilus.sshvpn.plist`