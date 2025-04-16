## **how to configure one with an **Auto Scaling Group (ASG)** in AWS using Terraform!**

---

## ⚖️ **What is a Load Balancer?**

A **Load Balancer** distributes incoming traffic across multiple targets (like EC2 instances) to improve:
- **Availability**
- **Scalability**
- **Fault Tolerance**

---

### 📦 Types of AWS Load Balancers

| Load Balancer | Use Case |
|---------------|----------|
| **ALB** (Application Load Balancer) | HTTP/HTTPS, Layer 7, Path-based routing |
| **NLB** (Network Load Balancer)     | TCP/UDP, Layer 4, Extreme performance |
| **CLB** (Classic)                   | Legacy, Layer 4/7 mix |

👉 We’ll focus on **ALB + Auto Scaling Group** — the most common web app setup.

---

## 🏗️ **Goal Architecture**

- Launch EC2s using **Auto Scaling Group**
- Place them behind an **Application Load Balancer**
- Use a **Launch Template** for instance config
- Automatically scale based on traffic

---

## 🛠️ Step-by-Step Terraform Setup

We'll use these files:

```
elb-setup/
├── main.tf
├── alb.tf
├── autoscaling.tf
├── launch_template.tf
├── security_groups.tf
├── outputs.tf
```

---

### 📄 `main.tf`

```hcl
provider "aws" {
  region = "us-east-1"
}
```

---

### 📄 `security_groups.tf`

```hcl
resource "aws_security_group" "alb_sg" {
  name   = "alb-sg"
  vpc_id = aws_vpc.main.id

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

resource "aws_security_group" "ec2_sg" {
  name   = "asg-ec2-sg"
  vpc_id = aws_vpc.main.id

  ingress {
    from_port       = 80
    to_port         = 80
    protocol        = "tcp"
    security_groups = [aws_security_group.alb_sg.id]
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

### 📄 `launch_template.tf`

```hcl
resource "aws_launch_template" "web" {
  name_prefix   = "web-template-"
  image_id      = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"

  user_data = base64encode(<<-EOF
              #!/bin/bash
              yum update -y
              yum install -y httpd
              systemctl start httpd
              echo "Hello from $(hostname)" > /var/www/html/index.html
              EOF
            )

  vpc_security_group_ids = [aws_security_group.ec2_sg.id]
  key_name               = aws_key_pair.my_key.key_name
}
```

---

### 📄 `alb.tf`

```hcl
resource "aws_lb" "alb" {
  name               = "web-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb_sg.id]
  subnets            = [aws_subnet.public.id, aws_subnet.public2.id] # 2 AZs
}

resource "aws_lb_target_group" "web_tg" {
  name     = "web-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id

  health_check {
    path                = "/"
    interval            = 30
    timeout             = 5
    healthy_threshold   = 3
    unhealthy_threshold = 2
    matcher             = "200"
  }
}

resource "aws_lb_listener" "web_listener" {
  load_balancer_arn = aws_lb.alb.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web_tg.arn
  }
}
```

---

### 📄 `autoscaling.tf`

```hcl
resource "aws_autoscaling_group" "web_asg" {
  name                      = "web-asg"
  desired_capacity          = 2
  max_size                  = 4
  min_size                  = 1
  vpc_zone_identifier       = [aws_subnet.public.id, aws_subnet.public2.id]
  target_group_arns         = [aws_lb_target_group.web_tg.arn]
  health_check_type         = "EC2"
  health_check_grace_period = 300

  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "ASG-Web"
    propagate_at_launch = true
  }
}
```

---

### 📄 `outputs.tf`

```hcl
output "alb_dns_name" {
  value = aws_lb.alb.dns_name
}
```

---

## ✅ Deploy it

1. Make sure the `aws_vpc.main`, `aws_subnet.public`, and `aws_subnet.public2` are defined.
2. Replace your `key_name` and `subnet_ids`.
3. Run:

```bash
terraform init
terraform apply
```

---

## 🧪 Test

Open the ALB DNS name in a browser:

```bash
http://<alb_dns_name>
```

You should see:
```
Hello from ip-10-0-x-x
```

Each refresh might show a different instance hostname!

**secure your Application Load Balancer (ALB) with HTTPS using AWS Certificate Manager (ACM)** and update your Terraform configuration accordingly.

---

## 🔐 Goal

We'll:
1. Request an SSL/TLS certificate via ACM
2. Add HTTPS listener to the ALB
3. Redirect HTTP (port 80) → HTTPS (port 443)
4. Use the certificate in the listener rule

---

## 📝 Prerequisites

- A **public domain name** you own (like `example.com`)
- The domain should be **verified in Route 53** or allow DNS/email validation

---

## 📦 Updated Terraform Files

### ✅ 1. `cert.tf` — ACM Certificate Request

```hcl
resource "aws_acm_certificate" "ssl_cert" {
  domain_name       = "yourdomain.com"            # Replace
  validation_method = "DNS"

  tags = {
    Name = "web-cert"
  }

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_route53_record" "cert_validation" {
  name    = aws_acm_certificate.ssl_cert.domain_validation_options[0].resource_record_name
  type    = aws_acm_certificate.ssl_cert.domain_validation_options[0].resource_record_type
  zone_id = "YOUR_ZONE_ID" # Replace with your hosted zone ID
  records = [aws_acm_certificate.ssl_cert.domain_validation_options[0].resource_record_value]
  ttl     = 60
}

resource "aws_acm_certificate_validation" "cert_validate" {
  certificate_arn         = aws_acm_certificate.ssl_cert.arn
  validation_record_fqdns = [aws_route53_record.cert_validation.fqdn]
}
```

---

### ✅ 2. Update `alb.tf` — Add HTTPS Listener

```hcl
resource "aws_lb_listener" "https" {
  load_balancer_arn = aws_lb.alb.arn
  port              = 443
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-2016-08"
  certificate_arn   = aws_acm_certificate.ssl_cert.arn
  depends_on        = [aws_acm_certificate_validation.cert_validate]

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web_tg.arn
  }
}

resource "aws_lb_listener" "http_redirect" {
  load_balancer_arn = aws_lb.alb.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type = "redirect"

    redirect {
      port        = "443"
      protocol    = "HTTPS"
      status_code = "HTTP_301"
    }
  }
}
```

---

### ✅ 3. (Optional) Update `outputs.tf`

```hcl
output "secure_alb_url" {
  value = "https://${aws_lb.alb.dns_name}"
}
```

---

## 🚀 Deploy It

```bash
terraform init
terraform apply
```

> 🕓 Wait a few minutes for ACM validation to complete (it’s automatic if DNS is set right).

---

## ✅ Test It

Open your browser:

```bash
https://yourdomain.com
```

You should see a secure padlock 🔒 with a valid SSL cert.
