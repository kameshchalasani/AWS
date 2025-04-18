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

