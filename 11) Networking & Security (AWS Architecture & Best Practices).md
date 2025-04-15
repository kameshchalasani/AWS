**custom VPC** 

---

## 🧱 What You’ll Build

A fully functional **custom VPC** setup with:
- A **VPC**
- **Public** and **Private Subnets**
- **Internet Gateway** (for public internet access)
- **NAT Gateway** (for outbound internet access from private subnets)
- **Route Tables**

---

## ✅ Step-by-Step Guide (Terraform Version)

Let’s break it down:

---

### 📁 `vpc.tf` – Full Custom VPC Setup

```hcl
provider "aws" {
  region = "us-east-1"
}

# 1. Create VPC
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags = { Name = "my-custom-vpc" }
}

# 2. Create Public Subnet
resource "aws_subnet" "public" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
  map_public_ip_on_launch = true
  tags = { Name = "public-subnet" }
}

# 3. Create Private Subnet
resource "aws_subnet" "private" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "us-east-1a"
  tags = { Name = "private-subnet" }
}

# 4. Create Internet Gateway
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
  tags = { Name = "my-igw" }
}

# 5. Public Route Table
resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = { Name = "public-rt" }
}

# 6. Associate Public Route Table
resource "aws_route_table_association" "public_assoc" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public_rt.id
}

# 7. Elastic IP for NAT
resource "aws_eip" "nat_eip" {
  domain = "vpc"
}

# 8. NAT Gateway in Public Subnet
resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = aws_subnet.public.id
  tags = { Name = "nat-gateway" }
}

# 9. Private Route Table
resource "aws_route_table" "private_rt" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat.id
  }

  tags = { Name = "private-rt" }
}

# 10. Associate Private Route Table
resource "aws_route_table_association" "private_assoc" {
  subnet_id      = aws_subnet.private.id
  route_table_id = aws_route_table.private_rt.id
}
```

---

## 🌐 What This Architecture Enables

| Subnet Type | Can Reach Internet? | Needs Public IP? | Uses |
|-------------|---------------------|------------------|------|
| Public      | ✅ Yes              | ✅ Yes           | Bastion, Load Balancer, Web Apps |
| Private     | ✅ Outbound only    | ❌ No            | App servers, RDS, Lambda |

---
Let’s **expand the VPC setup** to include **EC2 instances** in both subnets, apply **Security Groups**, and briefly touch on **NACLs** — all using **Terraform**.

---

## 🔧 Add EC2 Instances to VPC

We’ll add:
- One EC2 instance in the **public subnet** (for SSH/web access)
- One EC2 instance in the **private subnet** (can access internet via NAT)

---

### 📁 `instances.tf`

```hcl
# Key Pair (replace with your public key)
resource "aws_key_pair" "my_key" {
  key_name   = "my-key"
  public_key = file("~/.ssh/id_rsa.pub")
}

# Security Group for Public EC2
resource "aws_security_group" "public_sg" {
  name        = "public-sg"
  description = "Allow SSH and HTTP"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Public EC2 instance
resource "aws_instance" "public_ec2" {
  ami                    = "ami-0c02fb55956c7d316" # Amazon Linux 2 (us-east-1)
  instance_type          = "t2.micro"
  subnet_id              = aws_subnet.public.id
  associate_public_ip_address = true
  key_name               = aws_key_pair.my_key.key_name
  security_groups        = [aws_security_group.public_sg.id]

  tags = {
    Name = "Public-EC2"
  }
}

# Security Group for Private EC2
resource "aws_security_group" "private_sg" {
  name        = "private-sg"
  description = "Allow internal traffic"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port       = 0
    to_port         = 0
    protocol        = "-1"
    security_groups = [aws_security_group.public_sg.id] # Allow from public SG
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Private EC2 instance
resource "aws_instance" "private_ec2" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.private.id
  key_name      = aws_key_pair.my_key.key_name
  security_groups = [aws_security_group.private_sg.id]

  tags = {
    Name = "Private-EC2"
  }
}
```

---

## 🔐 Bonus: NACL Overview

| Feature                | Security Groups           | NACLs                            |
|------------------------|---------------------------|----------------------------------|
| **Stateful?**          | ✅ Yes                    | ❌ No (must allow inbound & outbound explicitly) |
| **Applies to?**        | ENIs (Instances)          | Subnets                         |
| **Common Use Case**    | Instance-level control     | Extra subnet-level filtering     |



---

## ✅ Ready to Deploy?

Once all files are in place, deploy with:

```bash
terraform init
terraform apply
Great! Let’s take this setup **one step further** and:

> 🎯 **Add a Bastion Host** in the **Public Subnet**  
> 🛡️ Access the **Private EC2 instance** **securely** via the Bastion using SSH  
> 🔐 Tighten security so no direct public access goes to the private instance  

---

## 💡 What is a Bastion Host?

A **bastion host** is a special-purpose EC2 instance in a **public subnet** that you **SSH into**, and then use it to reach **private instances** (in private subnets).

---

## 🧱 Updated Terraform Additions (Bastion + Tightened Access)

### ✅ `security_groups.tf`

```hcl
# Bastion SG - only SSH from your IP
resource "aws_security_group" "bastion_sg" {
  name        = "bastion-sg"
  description = "Allow SSH from trusted IP"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["YOUR_PUBLIC_IP/32"] # replace with your IP
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Update Private SG to only allow SSH from Bastion
resource "aws_security_group" "private_sg" {
  name        = "private-sg"
  description = "Allow SSH from bastion"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port       = 22
    to_port         = 22
    protocol        = "tcp"
    security_groups = [aws_security_group.bastion_sg.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

---

### ✅ `bastion.tf`

```hcl
resource "aws_instance" "bastion" {
  ami                         = "ami-0c02fb55956c7d316" # Amazon Linux 2 (us-east-1)
  instance_type               = "t2.micro"
  subnet_id                   = aws_subnet.public.id
  associate_public_ip_address = true
  key_name                    = aws_key_pair.my_key.key_name
  security_groups             = [aws_security_group.bastion_sg.id]

  tags = {
    Name = "Bastion-Host"
  }
}
```

---

## 🔑 SSH Flow

### Step 1: SSH into Bastion

```bash
ssh -i ~/.ssh/id_rsa ec2-user@<Bastion_Public_IP>
```

### Step 2: SSH into Private Instance **from inside Bastion**

```bash
ssh -i ~/.ssh/id_rsa ec2-user@<Private_EC2_IP>
```

✅ Ensure the private EC2’s SG **allows SSH from Bastion SG**, and both EC2s use the **same key pair**.

---


---

## ✅ Project Structure

```bash
aws-bastion-setup/
├── main.tf
├── vpc.tf
├── subnets.tf
├── internet_nat.tf
├── security_groups.tf
├── bastion.tf
├── instances.tf
```

---

## 🔧 1. `main.tf` – Provider Setup

```hcl
provider "aws" {
  region = "us-east-1"
}
```

---

## 🏗️ 2. `vpc.tf` – VPC Definition

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags = { Name = "secure-vpc" }
}
```

---

## 🌐 3. `subnets.tf` – Subnets

```hcl
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "us-east-1a"
  tags = { Name = "public-subnet" }
}

resource "aws_subnet" "private" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "us-east-1a"
  tags = { Name = "private-subnet" }
}
```

---

## 🌉 4. `internet_nat.tf` – IGW, NAT, and Routing

```hcl
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
  tags   = { Name = "igw" }
}

resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}

resource "aws_route_table_association" "public_assoc" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public_rt.id
}

resource "aws_eip" "nat_eip" {
  domain = "vpc"
}

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = aws_subnet.public.id
  tags          = { Name = "nat" }
}

resource "aws_route_table" "private_rt" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat.id
  }
}

resource "aws_route_table_association" "private_assoc" {
  subnet_id      = aws_subnet.private.id
  route_table_id = aws_route_table.private_rt.id
}
```

---

## 🔐 5. `security_groups.tf` – Bastion + Private SG

```hcl
resource "aws_security_group" "bastion_sg" {
  name   = "bastion-sg"
  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["YOUR_PUBLIC_IP/32"] # Replace!
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "private_sg" {
  name   = "private-sg"
  vpc_id = aws_vpc.main.id

  ingress {
    from_port       = 22
    to_port         = 22
    protocol        = "tcp"
    security_groups = [aws_security_group.bastion_sg.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

---

## 🛡️ 6. `bastion.tf` – Bastion Host

```hcl
resource "aws_key_pair" "my_key" {
  key_name   = "my-key"
  public_key = file("~/.ssh/id_rsa.pub")
}

resource "aws_instance" "bastion" {
  ami                         = "ami-0c02fb55956c7d316"
  instance_type               = "t2.micro"
  subnet_id                   = aws_subnet.public.id
  associate_public_ip_address = true
  key_name                    = aws_key_pair.my_key.key_name
  security_groups             = [aws_security_group.bastion_sg.id]
  tags = { Name = "bastion" }
}
```

---

## 🖥️ 7. `instances.tf` – Private EC2

```hcl
resource "aws_instance" "private_ec2" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.private.id
  key_name      = aws_key_pair.my_key.key_name
  security_groups = [aws_security_group.private_sg.id]
  tags = { Name = "private-ec2" }
}
```

---

## 🚀 Deployment Steps

1. Place all `.tf` files in the same folder.
2. Replace `"YOUR_PUBLIC_IP/32"` with your IP (from https://whatismyipaddress.com).
3. Run:

```bash
terraform init
terraform apply
```

---

## 🔐 SSH into Private EC2

1. Get the public IP of the **bastion host** from the AWS console or Terraform output.
2. SSH into Bastion:

```bash
ssh -i ~/.ssh/id_rsa ec2-user@<bastion_public_ip>
```

3. From inside Bastion, SSH into private EC2:

```bash
ssh -i ~/.ssh/id_rsa ec2-user@<private_ec2_ip>
```

