---
title: "AWS CLI Command Snippets"
date: 2024-02-29
dateUpdated: Last Modified
permalink: /posts/aws-cli-cheatsheet/
tags:
  - Snippets
layout: layouts/post.njk
---

## Installing AWS CLI in Windows using Chocolatey

Start Command Prompt as Administrator and run this command:
```
choco install awscli
```

## Using AWS CLI

Always use profiles, and leave the default profile not configured in order to prevent mistakes.

```powershell
# Create profiles
aws configure --profile profilename

# TODO: wip
```

## Using AWS S3 CLI to upload files to Backblaze B2 storage

Prerequisites you need to have ([see here more](https://www.backblaze.com/docs/cloud-storage-use-the-aws-cli-with-backblaze-b2)):
- Backblaze Key ID
- Backblaze App Key
- Backblaze S3 endpoint URL

Configure the `backblaze` profile like this:
```powershell
$FolderPathToBackup = $Config.FolderPathToBackup
$BackblazeBucketName = $Config.BackblazeBucketName
$BackblazeS3EndpointUrl = $Config.BackblazeS3EndpointUrl

# Configure AWS CLI with Backblaze
$AWS_ACCESS_KEY_ID = "<BackblazeKeyID>"
$AWS_SECRET_ACCESS_KEY = "<BackblazeAppKey>"

aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID --profile backblaze; `
    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY --profile backblaze
```

Test the integration:
```powershell
aws s3 ls --endpoint-url=$BackblazeS3EndpointUrl --profile backblaze
```

Upload the file to the S3-compatible bucket in Backblaze B2:
```powershell
aws s3 cp "C:/path/to/backup/" s3://custom-bucket-name-in-backblaze/ --recursive --profile backblaze --endpoint-url=$BackblazeS3EndpointUrl
```

## Additional Resources

- https://aws.amazon.com/cli/
- https://www.backblaze.com/docs/cloud-storage-use-the-aws-cli-with-backblaze-b2

