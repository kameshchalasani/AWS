Let's break down **Security Groups** and **Network ACLs (NACLs)** in AWS and walk through how to configure them to control traffic for **EC2** and **VPC**.

---

## 🔐 Security Groups vs NACLs – What's the Difference?

| Feature | Security Group | NACL |
|--------|----------------|------|
| **Type** | Virtual firewall for **instances** | Firewall for **subnets** |
| **Stateful** | ✅ Yes | ❌ No |
| **Rules** | Allow rules only | Allow & deny rules |
| **Applies to** | EC2 | All resources in the subnet |
| **Direction** | Inbound & Outbound | Inbound & Outbound separately |

---

## 🛠️ Step-by-Step: Configure Security Groups and NACLs

---

### 🧱 1. **Security Group** for EC2 (Allow SSH & HTTP)

#### ✅ Terraform Example

```hcl
resource "aws_security_group" "web_sg" {
  name        = "web-server-sg"
  description = "Allow SSH and HTTP"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"] # SSH
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"] # HTTP
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1" # all traffic
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

---

### 🧱 2. **Network ACL** for Subnet (Allow HTTP/HTTPS/SSH)

#### ✅ Terraform Example

```hcl
resource "aws_network_acl" "web_acl" {
  vpc_id = aws_vpc.main.id
  subnet_ids = [aws_subnet.public.id]

  ingress {
    rule_no    = 100
    protocol   = "6"
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 22
    to_port    = 22
  }

  ingress {
    rule_no    = 110
    protocol   = "6"
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 80
    to_port    = 80
  }

  egress {
    rule_no    = 100
    protocol   = "6"
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 0
    to_port    = 65535
  }

  tags = {
    Name = "WebACL"
  }
}
```

---

## 🧪 Test It

After associating:
- EC2 with `web_sg`
- Subnet with `web_acl`

You should be able to:
- SSH to the EC2 from your IP (port 22)
- Access a web server running on the EC2 via HTTP (port 80)

---

## ✅ Best Practices

- **Security Groups**: Use for fine-grained, instance-level control
- **NACLs**: Use for broader subnet-level protections (especially useful for deny rules)
- Deny **all traffic by default** in private subnets unless explicitly needed
- Use **NACLs to block specific IPs**, which SGs can't do

---

