---
title: How To Transfer Files To S3 Buckets Using SFTP and FTP
date: 2023-11-26T00:33:07.850Z
published: true
tags:
  - S3
  - SFTP
  - File transfer
cover_image: ../../static/images/uploads/s3fts.png
description: T﻿his post explains how to transfer files to and from AWS S3 using
  the S3 File Transfer Server. By using the S3 File Transfer Server, you can
  easily upload and download files in your S3 buckets using SFTP. This ensures
  data is always encrypted in-transit and assuming your bucket was set up with
  encryption enabled, data will be encrypted at-rest too.
---
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