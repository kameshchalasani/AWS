Let's break down **AWS Key Management Service (KMS)** — one of the core AWS security tools used for **encryption and decryption** of data at rest and in transit.

---

## 🔐 What is AWS KMS?

**AWS KMS** is a **managed encryption service** that allows you to:
- Create and manage **cryptographic keys**
- Encrypt and decrypt **data or AWS resources**
- Control access using **IAM policies, key policies, and grants**
- Integrate with **services like S3, EBS, RDS, Lambda, CloudTrail, Secrets Manager, etc.**

---

## 🎯 Use Case: Encrypt and Decrypt Sensitive Data

We’ll:
1. Create a KMS key
2. Use it to encrypt/decrypt a string
3. Optionally, use it with S3, EBS, or RDS

---

## 🛠️ 1. Create a Customer Master Key (CMK)

### 🔧 Terraform Example:

```hcl
resource "aws_kms_key" "my_key" {
  description             = "KMS key for encrypting sensitive app data"
  deletion_window_in_days = 10
  enable_key_rotation     = true
}

resource "aws_kms_alias" "my_key_alias" {
  name          = "alias/my-app-key"
  target_key_id = aws_kms_key.my_key.key_id
}
```

---

## 🛠️ 2. Encrypt and Decrypt Text Data via CLI

### 🔐 Encrypt a string

```bash
aws kms encrypt \
  --key-id alias/my-app-key \
  --plaintext "This is a secret" \
  --output text \
  --query CiphertextBlob
```

Output:
```
AQICAHg...base64...==
```

### 🔓 Decrypt it

```bash
aws kms decrypt \
  --ciphertext-blob fileb://<(echo "AQICAHg...==" | base64 --decode) \
  --output text \
  --query Plaintext | base64 --decode
```

Output:
```
This is a secret
```

> ⚠️ Only IAM principals with the correct key permissions can encrypt/decrypt data.

---

## 🧱 Common KMS Integrations

| Service | KMS Use |
|--------|---------|
| **S3** | Bucket or object-level encryption |
| **EBS** | Volume encryption |
| **RDS** | Encrypted DB instances |
| **Lambda** | Encrypt env variables |
| **Secrets Manager / SSM** | Automatically uses KMS keys |

---

## 🔒 KMS Access Control

KMS uses:
- **Key policies** – Resource-based access control on the KMS key
- **IAM policies** – Identity-based access control
- **Grants** – Temporary access given to services (like EC2 or Lambda)

### Example IAM Policy for KMS Usage

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:GenerateDataKey"
      ],
      "Resource": "*"
    }
  ]
}
Let's **encrypt S3 data using AWS KMS** — fully integrated and secure.

---

## 🎯 Goal

- Use a **KMS key** (CMK) to encrypt objects in an **S3 bucket**
- Upload and retrieve encrypted objects
- Enable **automatic key rotation** and **access control**

---

## 🛠️ Step-by-Step Guide

---

### 🔧 1. Create a KMS Key (Terraform)

```hcl
resource "aws_kms_key" "s3_kms_key" {
  description             = "KMS key for S3 bucket encryption"
  deletion_window_in_days = 10
  enable_key_rotation     = true
}

resource "aws_kms_alias" "s3_kms_alias" {
  name          = "alias/s3-bucket-key"
  target_key_id = aws_kms_key.s3_kms_key.key_id
}
```

---

### 🪣 2. Create an S3 Bucket with KMS Encryption Enabled

```hcl
resource "aws_s3_bucket" "secure_bucket" {
  bucket = "my-secure-s3-bucket-12345"
  force_destroy = true
}

resource "aws_s3_bucket_server_side_encryption_configuration" "sse_config" {
  bucket = aws_s3_bucket.secure_bucket.id

  rule {
    apply_server_side_encryption_by_default {
      kms_master_key_id = aws_kms_key.s3_kms_key.arn
      sse_algorithm     = "aws:kms"
    }
  }
}
```

---

### 👤 3. IAM Permissions to Use the KMS Key

Ensure the IAM user/role (e.g., EC2 or CLI user) has access to **both S3** and the **KMS key**.

#### ✅ Example IAM Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::my-secure-s3-bucket-12345/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:GenerateDataKey"
      ],
      "Resource": "arn:aws:kms:REGION:ACCOUNT_ID:key/YOUR_KMS_KEY_ID"
    }
  ]
}
```

Replace:
- `REGION`, `ACCOUNT_ID`, and `YOUR_KMS_KEY_ID` accordingly

---

### 📤 4. Upload Encrypted File to S3 via CLI

```bash
aws s3 cp myfile.txt s3://my-secure-s3-bucket-12345/ \
  --sse aws:kms \
  --sse-kms-key-id alias/s3-bucket-key
```

> You can also skip `--sse` if server-side encryption is **enabled by default** in bucket settings.

---

### 📥 5. Download and Verify

```bash
aws s3 cp s3://my-secure-s3-bucket-12345/myfile.txt .
```

You should receive the file decrypted automatically (if IAM permissions and KMS access are correct).

---

## ✅ Done! Your S3 objects are now encrypted using a custom KMS key.

let's extend this to a **secure serverless application** using **S3 + Lambda + KMS**, and I’ll also include a **CloudFormation version** for full automation. You’ll have both Infrastructure as Code approaches covered (Terraform + CloudFormation).

---

## 🔄 Scenario:
You're building a **serverless function** that reads an encrypted file from **S3**, decrypts it using **KMS**, and processes the content.

---

## ✅ What We'll Set Up:

1. **KMS key**  
2. **S3 bucket** with KMS encryption  
3. **Lambda function** with permissions to use the KMS key  
4. **IAM role** for Lambda  
5. **CloudFormation template** (YAML)

---

## 🧾 CloudFormation Template – S3 + Lambda + KMS (YAML)

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Serverless app with S3, Lambda, and KMS encryption

Resources:

  ### KMS Key
  MyKMSKey:
    Type: AWS::KMS::Key
    Properties:
      Description: KMS key for S3 & Lambda encryption
      Enabled: true
      EnableKeyRotation: true
      KeyPolicy:
        Version: "2012-10-17"
        Statement:
          - Sid: AllowAccountAccess
            Effect: Allow
            Principal:
              AWS: !Sub arn:aws:iam::${AWS::AccountId}:root
            Action: "kms:*"
            Resource: "*"

  MyKMSKeyAlias:
    Type: AWS::KMS::Alias
    Properties:
      AliasName: alias/lambda-s3-kms-key
      TargetKeyId: !Ref MyKMSKey

  ### S3 Bucket with Encryption
  MySecureBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub my-secure-bucket-${AWS::AccountId}
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: aws:kms
              KMSMasterKeyID: !Ref MyKMSKey

  ### IAM Role for Lambda
  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: lambda-s3-kms-role
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
      Policies:
        - PolicyName: S3KMSLambdaPolicy
          PolicyDocument:
            Version: "2012-10-17"
            Statement:
              - Effect: Allow
                Action:
                  - s3:GetObject
                Resource: !Sub arn:aws:s3:::${MySecureBucket}/*
              - Effect: Allow
                Action:
                  - kms:Decrypt
                Resource: !Ref MyKMSKey
              - Effect: Allow
                Action:
                  - logs:CreateLogGroup
                  - logs:CreateLogStream
                  - logs:PutLogEvents
                Resource: "*"

  ### Lambda Function
  MyLambdaFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: S3KMSReader
      Handler: index.handler
      Runtime: nodejs18.x
      Role: !GetAtt LambdaExecutionRole.Arn
      Timeout: 10
      Code:
        ZipFile: |
          const AWS = require('aws-sdk');
          const s3 = new AWS.S3();
          exports.handler = async (event) => {
              const bucket = process.env.BUCKET_NAME;
              const key = 'my-encrypted-file.txt';
              const data = await s3.getObject({ Bucket: bucket, Key: key }).promise();
              const content = data.Body.toString('utf-8');
              console.log('Decrypted content:', content);
              return { statusCode: 200, body: content };
          };
      Environment:
        Variables:
          BUCKET_NAME: !Ref MySecureBucket
```

---

## 🚀 Deploy It

1. Save the above template as `s3-kms-lambda.yaml`
2. Run the following CLI command:
   ```bash
   aws cloudformation deploy \
     --template-file s3-kms-lambda.yaml \
     --stack-name s3-kms-serverless \
     --capabilities CAPABILITY_NAMED_IAM
   ```

---

## 🧪 Test the Setup

1. Upload a file (e.g. `my-encrypted-file.txt`) to the bucket:
   ```bash
   aws s3 cp myfile.txt s3://my-secure-bucket-<account-id>/ --sse aws:kms --sse-kms-key-id alias/lambda-s3-kms-key
   ```

2. Invoke the Lambda manually or via test event in AWS Console.

---

## 🔒 Result

- Data in S3 is encrypted with **KMS**
- Lambda uses its IAM role and **KMS Decrypt** to securely read and log the content
- No plaintext access outside AWS-managed encryption flow
