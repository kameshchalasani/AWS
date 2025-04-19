Let's dive into **AWS IAM (Identity and Access Management)** — the security backbone of AWS.

---

## 🔐 What Is IAM?

IAM lets you **securely control access** to AWS services and resources for users, groups, and applications.

---

## 🎯 Goals

- ✅ Create IAM users, groups, roles, and policies
- ✅ Enforce **MFA (Multi-Factor Authentication)**
- ✅ Apply **Least Privilege Access** best practices

---

## 🧱 1. IAM Building Blocks

| Component | Description |
|----------|-------------|
| **User** | Represents an individual (with login credentials) |
| **Group** | Collection of users with shared permissions |
| **Role** | Set of permissions assumed by users, services, or apps |
| **Policy** | JSON document that defines what actions are allowed or denied |

---

## 🛠️ Hands-On with IAM in Terraform

### ✅ Create an IAM User, Group, and Attach Policy

```hcl
resource "aws_iam_group" "dev_team" {
  name = "Developers"
}

resource "aws_iam_user" "alice" {
  name = "alice"
}

resource "aws_iam_user_group_membership" "alice_group" {
  user = aws_iam_user.alice.name
  groups = [aws_iam_group.dev_team.name]
}

resource "aws_iam_policy" "readonly_policy" {
  name        = "ReadOnlyAccessPolicy"
  description = "Read-only access to AWS services"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = ["ec2:Describe*", "s3:List*", "s3:Get*"]
        Effect = "Allow"
        Resource = "*"
      }
    ]
  })
}

resource "aws_iam_group_policy_attachment" "attach_readonly" {
  group      = aws_iam_group.dev_team.name
  policy_arn = aws_iam_policy.readonly_policy.arn
}
```

---

### ✅ Implement MFA (Multi-Factor Authentication)

MFA cannot be enforced directly via Terraform, but you can **enforce MFA via a policy**:

```hcl
resource "aws_iam_policy" "mfa_enforce" {
  name = "MFAEnforcement"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid = "BlockAllUnlessMFAChecked"
        Effect = "Deny"
        Action = "*"
        Resource = "*"
        Condition = {
          BoolIfExists = {
            "aws:MultiFactorAuthPresent" = "false"
          }
        }
      }
    ]
  })
}

resource "aws_iam_user_policy_attachment" "mfa_attach" {
  user       = aws_iam_user.alice.name
  policy_arn = aws_iam_policy.mfa_enforce.arn
}
```

> 🧠 This policy denies all actions unless MFA is present in the request context.

---

## 🔐 Best Practices: Least Privilege Access

- Use **IAM roles** for EC2, Lambda, and other services instead of embedding credentials
- Create **custom policies** with only the needed actions and resources
- Rotate access keys regularly (or use short-lived tokens via roles)

---

Let's go hands-on with **IAM roles**, particularly for **EC2** to access **S3** with **least privilege** — and use temporary credentials via **AWS STS** (Security Token Service).

---

## 🎯 Goal

- Create an **IAM role** with **S3 read-only access**
- Attach it to an EC2 instance
- Use **temporary credentials** (automatically handled by EC2 metadata service)
- Verify access to S3 via the CLI on that EC2

---

## 🛠️ 1. Create IAM Role for EC2 with S3 Read-Only

```hcl
resource "aws_iam_role" "ec2_s3_readonly_role" {
  name = "EC2S3ReadOnlyRole"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
}

resource "aws_iam_policy_attachment" "attach_s3_readonly" {
  name       = "attach_s3_readonly"
  roles      = [aws_iam_role.ec2_s3_readonly_role.name]
  policy_arn = "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
}
```

---

## 🛠️ 2. Create an Instance Profile and Attach It to EC2

```hcl
resource "aws_iam_instance_profile" "ec2_instance_profile" {
  name = "EC2S3ReadOnlyProfile"
  role = aws_iam_role.ec2_s3_readonly_role.name
}

resource "aws_instance" "my_ec2" {
  ami           = "ami-0c55b159cbfafe1f0" # Amazon Linux 2 (replace if needed)
  instance_type = "t2.micro"
  key_name      = "your-key-pair"
  iam_instance_profile = aws_iam_instance_profile.ec2_instance_profile.name

  tags = {
    Name = "EC2-with-S3-Access"
  }
}
```

---

## 🧪 3. Verify Role Access on EC2

After the EC2 is launched:

1. Connect via SSH:
   ```bash
   ssh -i "your-key.pem" ec2-user@<EC2-Public-IP>
   ```

2. Run:
   ```bash
   aws sts get-caller-identity
   ```

   ✅ You should see an IAM role ARN instead of user credentials.

3. Try listing S3 buckets:
   ```bash
   aws s3 ls
   ```

   ✅ You’ll be able to list S3 buckets, but not modify them (due to read-only).

---

## 🔐 What's Happening Under the Hood?

- EC2 automatically uses the role’s **temporary security credentials** via the **metadata endpoint** at:
  ```
  http://169.254.169.254/latest/meta-data/iam/security-credentials/
  ```
- These rotate automatically and are scoped with **least privilege**

---

