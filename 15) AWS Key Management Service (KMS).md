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
