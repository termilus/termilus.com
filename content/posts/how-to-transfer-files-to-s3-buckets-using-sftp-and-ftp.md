---
title: How To Transfer Files To S3 Buckets Using SFTP and FTP
date: 2023-11-26T00:33:07.850Z
published: true
tags:
  - S3
  - SFTP
  - File transfer
  - Guide
cover_image: ../../static/images/uploads/s3fts.png
description: T﻿his post explains how to transfer files to and from AWS S3 using
  the S3 File Transfer Server. By using the S3 File Transfer Server, you can
  easily upload and download files in your S3 buckets using SFTP. This ensures
  data is always encrypted in-transit and assuming your bucket was set up with
  encryption enabled, data will be encrypted at-rest too.
---
## E﻿stimated Deployment Time

1﻿0 Minutes

## W﻿hat

T﻿he S3 File Transfer Server is a fast, secure, and super simple appliance to make getting files into and out of S3 a breeze. Deploying the S3 File Transfer Server, allows you to expose SFTP to your clients or other internal and external services. Files are uploaded and downloaded in real-time and never persist on the appliance itself. 

The server can be encrypted at deployment and all communication can be over SSH and SFTP, meaning all traffic is encrypted at all times. FTP is supported but not recommended for production deployments.

T﻿he S3 File Transfer Server was designed to be deployed quickly and easily. Configuration is straight forward and maintenance is automated.

## W﻿hy

* T﻿he S3 File Transfer Server is a convenient, familiar file transfer capabilities with S3 scalability and durability.
* T﻿he appliance is a turn-key solution, ready for production deployments.
* E﻿ncryption and speed are baked into the appliance to protect sensitive data while enabling speedy file transfers.

## H﻿ow

D﻿eploying, setting up and configuring the S3 File Transfer Server is super quick and easy. Just follow these steps.

1. To begin, you will first need to deploy the [S3 File Transfer Server](https://aws.amazon.com/marketplace/pp/prodview-shh2f5imxqqm6) from the AWS Marketplace.
2. N﻿ext, configure tags on the EC2 instance. The tag name should be "Buckets" and the value should be one or more S3 bucket names that will be used for file transfers.

   ![](../../static/images/uploads/tags1.png "At least one bucket must be specified")

   ![](../../static/images/uploads/tags2.png "Or, multiple buckets may be specified")
3. N﻿ext, in IAM, create a new policy called `S3-File-Transfer-Server-Policy`. In this policy, paste the following JSON:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1473154086000",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeTags"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": "arn:aws:s3:::*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:AbortMultipartUpload",
                "s3:DeleteObject",
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::*/*"
        }
    ]
}
```

4. Now, create an IAM role called `S3-FileTransfer-Server-Role` and attach the `S3-File-Transfer-Server-Policy` that we just created.

   ![](../../static/images/uploads/role.png "Attaching the policy to the role")
5. Next, modify the role of the EC2 instance so it has the S3-File-Transfer-Server-Role attached to it.

   ![](../../static/images/uploads/modifyiamrole1.png)

   ![](../../static/images/uploads/modifyiamrole2.png)
6. Finally, now that the role has been created and attached to the EC2 instance, we just need to reboot the appliance.

   ![](../../static/images/uploads/reboot.png)

7﻿. And that's it! Now, you can now SFTP into the server:

   ![](../../static/images/uploads/sftp.png)

## T﻿roubleshooting

* U﻿nable to access the SFTP service?

  * D﻿ouble check the security group associated with the S3 File Transfer Server to ensure inbound traffic is permitted on port 22 from the appropriate source IP address.
* N﻿ot seeing files on the S3 File Transfer Server that exist in the bucket?

  * M﻿ake sure that the security policy is attached to the role and that the S3 File Transfer Server has the role attached to itself.



## F﻿AQ

* Does the S3 File Transfer Server appliance support single-AZ, multi-AZ or multi-region deployments? The appliance is a single EC2 instance that can be deployed into any VPC in any region.
* D﻿oes the S3 File Transfer Server appliance support all regions? Yes, all regions are supported.
* S﻿hould we use the root user for deploying the S3 File Transfer Server appliance? No, it is recommended to use a non-root user to deploy the appliance.
* S﻿hould the S3 File Transfer Server appliance be encrypted when deployed? Yes, security best-practice is to encrypt the appliance when provisioned.
* I﻿s data in-transit encrypted? Yes, communication via SSH and SFTP is encrypted in-transit.
* W﻿hich services used by S3 File Transfer Server are billable? The S3 File Transfer Server appliance incurs S3, EC2, and software cost when deployed. For example, a t3.medium S3 File Transfer Server instance costs $0.042 per hour plus $0.088 per hour for software for a total of $0.13 per hour of deployment.
* H﻿ow can I monitor the health of the S3 File Transfer Server appliance? Check the health of your server by viewing its status in the "Instance State" column in the EC2 Dashboard.
* H﻿ow does the S3 File Transfer Server appliance get patched and updated? The appliance is set to auto-install security patches on a daily basis.
* H﻿ow do I handle a non-responsive S3 File Transfer Server appliance? This is very rare, but a reboot should resolve the issue.
* Is there technical assistance available to help troubleshoot? Yes, we're available to assist and will typically respond within 1 business day. We can be reached by emailing: support at termilus.com