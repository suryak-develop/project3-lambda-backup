# 🤖 Automated S3 Backup System with Boto3 + Lambda

![AWS](https://img.shields.io/badge/AWS-Cloud-orange?logo=amazonaws)
![Lambda](https://img.shields.io/badge/Lambda-Serverless-yellow?logo=awslambda)
![Python](https://img.shields.io/badge/Python-Boto3-blue?logo=python)
![S3](https://img.shields.io/badge/S3-Storage-green?logo=amazons3)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

---

## 📋 Project Overview

This project demonstrates how to build a **fully automated, serverless backup system** using AWS Lambda and Python (Boto3) that copies files to S3 on a scheduled basis — with zero manual intervention.

---

## 🚨 Problem

Manual backups are time-consuming, error-prone, and easy to forget. A missed backup can mean permanent data loss. Businesses need reliable, automated backup solutions that run consistently.

---

## ✅ Solution

Built a serverless automated backup pipeline using:

| AWS Service | Purpose |
|-------------|---------|
| **S3** | Store backup files with versioning enabled |
| **Lambda** | Python (Boto3) function to automate backup logic |
| **CloudWatch** | Schedule Lambda using EventBridge cron rules |
| **IAM** | Least-privilege execution role for Lambda |

---

## 🏆 Result

- ✅ **Fully automated** daily backups — no manual steps
- ✅ **S3 Versioning** keeps multiple backup copies safe
- ✅ **Serverless** — no servers to manage or pay for when idle
- ✅ **Least-privilege IAM** role for maximum security
- ✅ CloudWatch logs every backup run for monitoring

---

## 🏗️ Architecture Diagram

```
CloudWatch Events (Cron Schedule)
         │
         │  Triggers daily at midnight
         ▼
   AWS Lambda Function
   (Python + Boto3)
         │
         │  Copies files to backup bucket
         ▼
   S3 Backup Bucket
   (Versioning Enabled)
         │
   IAM Role (Least Privilege)
   CloudWatch Logs (Monitoring)
```

---

## 🐍 Lambda Function Code

```python
import boto3
import os
from datetime import datetime

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    
    source_bucket = os.environ['SOURCE_BUCKET']
    backup_bucket = os.environ['BACKUP_BUCKET']
    timestamp = datetime.now().strftime('%Y-%m-%d_%H-%M-%S')
    
    # List all objects in source bucket
    response = s3.list_objects_v2(Bucket=source_bucket)
    
    if 'Contents' not in response:
        print("No files found to backup.")
        return {'statusCode': 200, 'body': 'No files to backup'}
    
    backed_up = 0
    for obj in response['Contents']:
        source_key = obj['Key']
        backup_key = f"backups/{timestamp}/{source_key}"
        
        # Copy object to backup bucket
        s3.copy_object(
            CopySource={'Bucket': source_bucket, 'Key': source_key},
            Bucket=backup_bucket,
            Key=backup_key
        )
        backed_up += 1
        print(f"Backed up: {source_key} → {backup_key}")
    
    print(f"Backup complete! {backed_up} files backed up at {timestamp}")
    return {
        'statusCode': 200,
        'body': f'Successfully backed up {backed_up} files'
    }
```

---

## 🛠️ Step-by-Step Setup

### Step 1: Create S3 Buckets
```bash
# Source bucket (files to back up)
aws s3 mb s3://my-source-bucket

# Backup bucket with versioning
aws s3 mb s3://my-backup-bucket
aws s3api put-bucket-versioning \
  --bucket my-backup-bucket \
  --versioning-configuration Status=Enabled
```

### Step 2: Create IAM Role for Lambda
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::my-source-bucket/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-backup-bucket/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "*"
    }
  ]
}
```

### Step 3: Deploy Lambda Function
```bash
# Zip your Python file
zip function.zip lambda_function.py

# Create Lambda function
aws lambda create-function \
  --function-name s3-backup-function \
  --runtime python3.11 \
  --role arn:aws:iam::YOUR_ACCOUNT:role/lambda-s3-backup-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip \
  --environment Variables="{SOURCE_BUCKET=my-source-bucket,BACKUP_BUCKET=my-backup-bucket}"
```

### Step 4: Schedule with CloudWatch Events
```bash
# Create daily schedule (runs every day at midnight UTC)
aws events put-rule \
  --name daily-backup-rule \
  --schedule-expression "cron(0 0 * * ? *)" \
  --state ENABLED

# Add Lambda as target
aws events put-targets \
  --rule daily-backup-rule \
  --targets "Id=1,Arn=arn:aws:lambda:YOUR_REGION:YOUR_ACCOUNT:function:s3-backup-function"
```

---

## 📁 Project Files

```
project3-lambda-backup/
├── lambda_function.py       # Main backup Lambda code
├── iam-policy.json          # IAM least-privilege policy
├── requirements.txt         # Python dependencies (boto3)
└── README.md
```

---

## 💡 Key Learnings

- How to write AWS Lambda functions using Python and Boto3
- How S3 versioning protects against accidental data loss
- How CloudWatch EventBridge schedules automated tasks (cron)
- How to apply least-privilege IAM policies to Lambda functions
- How serverless architecture eliminates server management

---

## 🔗 Resources

- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/)
- [Boto3 S3 Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html)
- [CloudWatch Events / EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/)
- [S3 Versioning Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html)

---

*Built as part of the AWS re/Start Program* ☁️
