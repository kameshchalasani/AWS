

## 🛠️ **Step-by-Step: Set up a MySQL/PostgreSQL RDS Database**

---

### 📍 Step 1: Launch an RDS Instance

1. Go to **RDS Console**: https://console.aws.amazon.com/rds/
2. Click **“Create database”**
3. Select **Standard Create**
4. **Engine options**: Choose **MySQL** or **PostgreSQL**
5. **Templates**: Choose **Free tier** (for testing)
6. Configure:
   - DB instance identifier: `mydb-instance`
   - Master username: `admin`
   - Password: `yourpassword123` (store safely)
7. DB instance size: `db.t3.micro`
8. Storage: default is fine for now (20 GiB)
9. **Connectivity**:
   - VPC: default
   - Public access: **Yes** (if you want to connect from your local machine)
   - VPC security group: Create new, allow port **3306 for MySQL** or **5432 for PostgreSQL**
10. Click **Create database**

Wait a few minutes while AWS provisions your instance.

---

### 🔓 Step 2: Modify Inbound Security Rules

1. Go to **EC2 → Security Groups**
2. Find the group assigned to your RDS instance
3. Add an **Inbound Rule**:
   - Type: MySQL/Aurora (3306) or PostgreSQL (5432)
   - Source: Your IP or `0.0.0.0/0` (for testing only)

---

### 📡 Step 3: Connect to the Database

#### ✅ From Your Local Machine (via MySQL/PostgreSQL client):

Get the **endpoint** from the RDS dashboard.

**MySQL Example:**
```bash
mysql -h your-db-endpoint.rds.amazonaws.com -u admin -p
```

**PostgreSQL Example:**
```bash
psql -h your-db-endpoint.rds.amazonaws.com -U admin -d postgres
```

---

### 🔗 Step 4: Connect RDS to an Application (Example with EC2)

Let’s say your app is running on an EC2 instance (Node.js, PHP, Python, etc.):

```env
DB_HOST=your-db-endpoint.rds.amazonaws.com
DB_USER=admin
DB_PASS=yourpassword123
DB_NAME=mydatabase
```

In code (Node.js example using MySQL):

```js
const mysql = require('mysql2');
const connection = mysql.createConnection({
  host: 'your-db-endpoint.rds.amazonaws.com',
  user: 'admin',
  password: 'yourpassword123',
  database: 'mydatabase'
});
```

✅ Make sure your **EC2 instance and RDS are in the same VPC** or the security group allows connections between them.

---

## 🧪 Optional Extras:

- Set up **RDS backups & snapshots**
- Enable **Multi-AZ for high availability**
- Enable **monitoring & alerts (CloudWatch)**



## ✅ **Terraform – RDS MySQL Example**

### 📁 `main.tf`

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_db_subnet_group" "default" {
  name       = "my-db-subnet-group"
  subnet_ids = ["subnet-xxxxxx", "subnet-yyyyyy"] # Replace with your real subnet IDs

  tags = {
    Name = "MyDBSubnetGroup"
  }
}

resource "aws_security_group" "rds_sg" {
  name        = "rds-access"
  description = "Allow DB access"
  vpc_id      = "vpc-xxxxxxxx" # Replace with your VPC ID

  ingress {
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Use cautiously; ideally use your IP
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_db_instance" "mysql" {
  identifier              = "mydb-instance"
  engine                  = "mysql"
  instance_class          = "db.t3.micro"
  allocated_storage       = 20
  db_name                 = "mydatabase"
  username                = "admin"
  password                = "password1234"
  publicly_accessible     = true
  skip_final_snapshot     = true
  vpc_security_group_ids  = [aws_security_group.rds_sg.id]
  db_subnet_group_name    = aws_db_subnet_group.default.name
}
```

> 🔧 Replace subnet/VPC IDs and secure your credentials before production.

---

## ✅ **CloudFormation – RDS MySQL**

### 📁 `rds-mysql.yaml`

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: MySQL RDS Setup with Public Access

Resources:
  RDSMySQLSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow MySQL Access
      VpcId: vpc-xxxxxxxx  # Replace with your VPC ID
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          CidrIp: 0.0.0.0/0  # Open access for demo; restrict in production

  MyDBInstance:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceIdentifier: mydb-instance
      DBInstanceClass: db.t3.micro
      Engine: mysql
      MasterUsername: admin
      MasterUserPassword: password1234
      AllocatedStorage: 20
      DBName: mydatabase
      PubliclyAccessible: true
      VPCSecurityGroups:
        - !GetAtt RDSMySQLSecurityGroup.GroupId
```

> 💡 Deploy with:

```bash
aws cloudformation deploy \
  --template-file rds-mysql.yaml \
  --stack-name RDSMySQLStack \
  --capabilities CAPABILITY_NAMED_IAM


## ✅ What We’re Building

- 1x EC2 instance (Amazon Linux 2)
- 1x RDS MySQL instance
- Security group allowing EC2 ↔ RDS communication
- Optionally, install MySQL client on EC2 and connect at launch

---

## 🚀 Terraform: EC2 + RDS Integration

### 📁 `main.tf`

```hcl
provider "aws" {
  region = "us-east-1"
}

# Security group for EC2 and RDS
resource "aws_security_group" "ec2_rds_sg" {
  name        = "ec2-rds-sg"
  description = "Allow EC2 to access RDS"
  vpc_id      = "vpc-xxxxxxxx"  # Replace with your VPC ID

  ingress {
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"] # Limit this in prod
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["YOUR_PUBLIC_IP/32"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_db_subnet_group" "default" {
  name       = "my-db-subnet-group"
  subnet_ids = ["subnet-xxxxx", "subnet-yyyyy"] # Replace with real subnet IDs
}

# RDS Instance
resource "aws_db_instance" "mysql" {
  identifier              = "mydb-instance"
  engine                  = "mysql"
  instance_class          = "db.t3.micro"
  allocated_storage       = 20
  db_name                 = "mydatabase"
  username                = "admin"
  password                = "password1234"
  publicly_accessible     = true
  skip_final_snapshot     = true
  vpc_security_group_ids  = [aws_security_group.ec2_rds_sg.id]
  db_subnet_group_name    = aws_db_subnet_group.default.name
}

# EC2 Instance
resource "aws_instance" "app" {
  ami           = "ami-0c02fb55956c7d316" # Amazon Linux 2
  instance_type = "t2.micro"
  key_name      = "your-key-pair"  # Replace with your key

  vpc_security_group_ids = [aws_security_group.ec2_rds_sg.id]
  subnet_id              = "subnet-xxxxx" # Replace

  user_data = <<-EOF
              #!/bin/bash
              yum update -y
              yum install -y mysql
              echo '[INFO] Connecting to RDS...'
              mysql -h ${aws_db_instance.mysql.address} -u admin -ppassword1234 -e "SHOW DATABASES;"
              EOF

  tags = {
    Name = "EC2WithRDSConnection"
  }
}
```

---

## 🚀 CloudFormation: EC2 + RDS Integration

### 📁 `ec2-rds.yaml`

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: EC2 connected to RDS MySQL

Resources:

  SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow MySQL and SSH
      VpcId: vpc-xxxxxxxx
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: YOUR_PUBLIC_IP/32

  RDSInstance:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceIdentifier: mydb
      AllocatedStorage: 20
      DBInstanceClass: db.t3.micro
      Engine: mysql
      MasterUsername: admin
      MasterUserPassword: password1234
      DBName: mydatabase
      VPCSecurityGroups: 
        - !Ref SecurityGroup
      PubliclyAccessible: true

  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-0c02fb55956c7d316
      KeyName: your-key-pair
      SecurityGroupIds:
        - !Ref SecurityGroup
      SubnetId: subnet-xxxxx
      UserData:
        Fn::Base64: |
          #!/bin/bash
          yum update -y
          yum install -y mysql
          mysql -h mydb.xxxxxxxxxxxx.us-east-1.rds.amazonaws.com -u admin -ppassword1234 -e "SHOW DATABASES;"
```

---

## ✅ What Happens

- An EC2 instance is provisioned and installs MySQL client
- The instance connects to the RDS MySQL DB using `user_data`
- You can SSH into EC2 and manually run more queries

---
