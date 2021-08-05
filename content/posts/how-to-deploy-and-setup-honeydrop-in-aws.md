---
title: How to Deploy and Setup HoneyDrop in AWS
date: 2021-08-04T11:43:45.270Z
published: true
tags:
  - honeypot
  - honeydrop
  - deception
cover_image: ../../static/images/uploads/honeydroplogo.png
description: This guide details how to deploy and setup the HoneyDrop appliance
  in AWS. This appliance will expose fake services often targeted by cyber
  criminals. Its purpose is to lure attackers to itself thereby generating logs
  and alarms indicating that a nefarious actor may be on the network. When a
  HoneyDrop service is interacted with, the activity gets logged to AWS
  CloudWatch which can generate alerts to be sent to email or SMS.
  Alternatively, CloudWatch alarms can trigger Lambda functions to invoke
  incident response actions on your behalf.
---
## What

The HoneyDrop appliance is a low-interaction honeypot meant to be deployed in subnets where critical visibility of malicious activities is needed. THe HoneyDrop can be configured to emulate:

* Windows SQL servers
* Windows web servers
* Linux MySQL servers
* Linux web servers

When the HoneyDrop appliance is interacted with, logs are generated and sent to AWS CloudWatch within seconds for alerting. These alerts can trigger SNS notifications to send emails, SMS messages or launch Lambda functions.

## Why

* Gain visibility into your AWS VPS network environments.
* Enhance security posture without needing to install agents on instances.
* Lure attackers away from high value systems long enough to be identified.
* Enable automated incident response actions to isolate and quarantine compromised hosts.

## How

The following steps will explain how to setup and deploy the HoneyDrop appliance.

1. Deploy the HoneyDrop from the AWS marketplace into your target subnet.
2. Navigate to https://console.aws.amazon.com/iamv2/home#/roles and click on "Create role".
3. Set the trusted entity type to "AWS service" and the use case to "EC2", now click next.
4. Attach the "CloudWatchAgentServerPolicy" and click next.
5. Optionally add your keys and click next.
6. Name the role "HoneyDropRole" and click "Create role".
7. Navigate back to EC2, https://console.aws.amazon.com/ec2/v2/home#Instances:instanceState=running
8. Right-click on the HoneyDrop appliance, under "Security" click "Modify IAM role"
9. Select the HoneyDropRole and click "Save".
10. Now, SSH into the HoneyDrop appliance and update the honeydrop.config file: nano /etc/honeydrop/honeydrop.conf

    * To emulate a Windows SQL server, change the following services to true:

      * smb.enabled
      * mssql.enabled
    * To emulate a Windows web server, change the following services to true:

      * smb.enabled
      * http.enabled
      * httpproxy.enabled
    * To emulate a Linux MySQL server, change the following services to true:

      * mysql.enabled
      * ssh.enabled
      * vnc.enabled
    * To emulate a Linux web server, change the following services to true:

      * ftp.enabled
      * http.enabled
      * ssh.enabled
      * vnc.enabled
      * telnet.enabled
11. Finish editing the HoneyDrop configuration file, save it and reboot the appliance.

And that's it! Your appliance should now be up and running in your environment. Test the services you enabled by interacting with them and see the alerts appear in AWS CloudWatch.