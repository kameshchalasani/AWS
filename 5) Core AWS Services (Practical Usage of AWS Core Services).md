
## ✅ **What is EC2?**

**Amazon EC2 (Elastic Compute Cloud)** lets you launch **virtual servers (instances)** in the cloud. You can choose OS types (Linux, Windows), configure hardware, secure access, and scale based on demand.

---

## 🧪 **Hands-On Lab: EC2 from Start to AMI**

---

### 🟢 **1. Launch an EC2 Instance (Linux/Windows)**

#### **🔹 Steps:**
1. Go to the **[EC2 Console](https://console.aws.amazon.com/ec2/)**.
2. Click **“Launch instance”**.
3. Fill out the following:
   - **Name**: e.g., `MyLinuxInstance`
   - **Amazon Machine Image (AMI)**: Choose an OS:
     - `Amazon Linux 2` (Free Tier eligible)
     - `Microsoft Windows Server 2019` (for Windows)
   - **Instance type**: Select `t2.micro` (Free Tier eligible)
   - **Key pair (login)**: Create a new key pair or use an existing one
     - Save the `.pem` file safely (you’ll need it for SSH)
   - **Network settings**:
     - Select **Allow SSH (port 22)** for Linux
     - Select **Allow RDP (port 3389)** for Windows
   - **Storage**: Default 8GB is fine

4. Click **“Launch instance”**

✅ **Success Check**: You should see “Instance state: Running” in your EC2 dashboard.

---

### 🟢 **2. Connect to the EC2 Instance via SSH (Linux) or RDP (Windows)**

#### **🔹 Linux Instance (SSH):**
1. Open Terminal or use **EC2 Instance Connect** in the console.
2. Use this command:
```bash
ssh -i /path/to/your-key.pem ec2-user@your-ec2-public-ip
```
- Make sure the `.pem` file has the correct permission:
```bash
chmod 400 your-key.pem
```

#### **🔹 Windows Instance (RDP):**
1. From the EC2 dashboard, select the Windows instance → Click **Connect** → Choose **RDP Client**
2. Download the **Remote Desktop File**
3. Retrieve the **Administrator password** using your key pair
4. Open the `.rdp` file and enter the password

✅ **Success Check**: You’re now connected to your EC2 instance!

---

### 🟢 **3. Configure Security Groups and Key Pairs**

#### **🔹 Security Groups:**
- Think of these like **firewall rules**.
- You can modify inbound/outbound rules from the EC2 dashboard.
  - Allow specific IPs (e.g., only allow SSH/RDP from your IP)
  - Add custom ports (e.g., port 80 for web apps)

#### **🔹 Key Pairs:**
- Used for **secure access**.
- You can:
  - Create new key pairs from **EC2 → Key Pairs**
  - Use `.pem` for Linux or `.ppk` (converted using PuTTYgen) for Windows

---

### 🟢 **4. Create and Use AMIs (Amazon Machine Images)**

📌 **Why?** To take a snapshot of your configured instance for backups or replicating environments.

#### **🔹 Steps to Create an AMI:**
1. Go to **Instances → Select your instance**
2. Click **Actions → Image and templates → Create image**
3. Enter:
   - **Image name**: e.g., `MyAppImage`
   - (Optional) Add a description
4. Click **Create image**

AWS will create an AMI listed under **Images → AMIs**

#### **🔹 Launch New Instance from AMI:**
1. Go to **AMIs**
2. Select your image → Click **Launch instance from image**
3. Follow the same steps as launching a normal EC2 instance

✅ **Success Check**: You have a **custom EC2 instance** launched from your own image!

---

## ✅ Summary

| Task                              | You Learned                      |
|----------------------------------|----------------------------------|
| Launch EC2 Instance              | Linux or Windows (Free Tier)     |
| Connect to Instance              | SSH for Linux, RDP for Windows   |
| Configure Security & Access      | Key pairs and Security Groups    |
| Create & Use AMIs                | Backup or replicate environments |

---

🔥 **Next Step Ideas:**
- Host a web app (e.g., Node.js, Django, WordPress) on EC2
- Use EC2 with **RDS** or **Elastic Load Balancer**
- Schedule automatic EC2 backups with **Amazon Data Lifecycle Manager**
Awesome! Let's go with a **step-by-step guide to host a WordPress website on AWS EC2**, using **RDS (for the database)** and **Apache or Nginx (for the web server)**. This is a great hands-on project that covers key AWS services.  

---

# 🛠️ **Host a WordPress Website on AWS EC2 + RDS**

---

## ✅ **Overview**

We’ll set up:
- ✅ An **EC2 instance** (Linux) to host WordPress
- ✅ An **RDS MySQL database** for storing content
- ✅ **Security groups**, **Apache**, **PHP**, and **WordPress config**

---

## 🧩 **Step 1: Launch an RDS MySQL Instance**

📍 _This will be the WordPress database._

### 🔹 Steps:
1. Go to **RDS Console** → Click **Create Database**
2. Select:
   - **Standard Create**
   - **Engine:** MySQL
   - **Version:** Choose the default or latest
3. **DB Instance Settings**:
   - DB Identifier: `wordpress-db`
   - Master username: `admin`
   - Master password: (Set your own)
4. **DB Instance Class**: Choose `db.t3.micro` (Free Tier eligible)
5. **Storage**: 20GB (default is fine)
6. **Connectivity**:
   - VPC: Default
   - Public access: **Yes**
   - VPC security group: Create a new one or use default
7. **Database name**: `wordpress`

👉 Click **Create database**

✅ Success Check: Wait until the status becomes **Available**

---

## 🧩 **Step 2: Launch an EC2 Instance (Amazon Linux 2)**

📍 _This will host your WordPress files._

### 🔹 Steps:
1. Go to **EC2 Console** → Click **Launch Instance**
2. Choose:
   - **Name**: `wordpress-server`
   - **AMI**: Amazon Linux 2
   - **Instance Type**: `t2.micro` (Free Tier)
   - **Key Pair**: Create or use an existing key pair
3. **Security Group Settings**:
   - Allow:  
     - HTTP (port 80)  
     - HTTPS (port 443)  
     - SSH (port 22) — your IP only
4. Click **Launch Instance**

✅ Success Check: Once it's running, copy the **public IP**

---

## 🧩 **Step 3: Connect to EC2 and Set Up Apache, PHP, and WordPress**

### 🔹 Connect to EC2:
```bash
ssh -i your-key.pem ec2-user@your-ec2-public-ip
```

### 🔹 Install LAMP stack:
```bash
sudo yum update -y
sudo yum install -y httpd mariadb-server php php-mysqlnd php-fpm php-json wget unzip
```

### 🔹 Start and enable Apache:
```bash
sudo systemctl start httpd
sudo systemctl enable httpd
```

---

## 🧩 **Step 4: Download and Configure WordPress**

### 🔹 Download WordPress:
```bash
cd /var/www/html
sudo wget https://wordpress.org/latest.zip
sudo unzip latest.zip
sudo cp -r wordpress/* .
sudo rm -rf wordpress latest.zip
```

### 🔹 Set Permissions:
```bash
sudo chown -R apache:apache /var/www/html
sudo chmod -R 755 /var/www/html
```

---

## 🧩 **Step 5: Configure WordPress to Connect to RDS**

1. Rename the config file:
```bash
sudo mv wp-config-sample.php wp-config.php
```

2. Edit the file:
```bash
sudo nano wp-config.php
```

3. Update these values:
```php
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'admin' );
define( 'DB_PASSWORD', 'your-rds-password' );
define( 'DB_HOST', 'your-rds-endpoint' );
```

*(You’ll find the RDS endpoint in the RDS console → "Connectivity & security")*

---

## 🧩 **Step 6: Complete WordPress Setup via Web**

1. In your browser, go to:  
   `http://your-ec2-public-ip`

2. You’ll see the **WordPress installation screen**:
   - Choose language
   - Set site title, admin user, password, and email

3. Click **Install WordPress**

✅ Success Check: You’ll see the WordPress Dashboard at `http://your-ec2-public-ip/wp-admin`

---

## ✅ Done! Your WordPress site is live 🎉

---

## 🚀 What’s Next?

- 🌐 **Point a domain** to your EC2 IP using Route 53
- 🔐 **Secure with SSL** (using Let’s Encrypt)
- 💾 **Create AMIs and RDS snapshots** for backup
- 📈 **Enable Auto Scaling and Load Balancer**
Perfect! Let’s take your setup to the next level and **add a custom domain + SSL certificate** using **AWS Route 53 and AWS Certificate Manager (ACM)**. After that, we’ll look at how to **automate the entire infrastructure using CloudFormation or Terraform**.

---

## 🌐 **Step-by-Step: Connect a Custom Domain + Add SSL**

---

### 🧩 **1. Register a Domain or Use an Existing One**

You can either:
- Register a domain using **[Route 53 Domains](https://console.aws.amazon.com/route53/home#domain-registration:)**
- Or, use a domain you bought elsewhere (e.g., GoDaddy, Namecheap)

> ✅ If you already have a domain, you’ll point its DNS to Route 53 later.

---

### 🧩 **2. Set Up a Hosted Zone in Route 53**

1. Go to **Route 53** → **Hosted Zones** → Click **“Create Hosted Zone”**
2. Enter:
   - Domain name: `yourdomain.com`
   - Type: **Public Hosted Zone**
3. Click **Create**

✅ Route 53 will give you **Name Servers (NS records)**.

> 🔁 If using an external registrar:  
   - Go to your domain registrar → Update the **NS records** to match what Route 53 gives you.

---

### 🧩 **3. Create a Record Set (A Record) for EC2**

1. In your hosted zone, click **“Create Record”**
2. Type: `A – IPv4 address`
3. Name: Leave blank for root domain or put `www` for subdomain
4. Value: Enter **EC2 public IP**
5. Routing policy: Simple → Click **Create**

✅ Now `http://yourdomain.com` will point to your EC2 instance!

---

## 🔒 **Step-by-Step: Enable Free SSL with AWS Certificate Manager + Load Balancer**

---

### 🧩 **4. Request an SSL Certificate (ACM)**

1. Go to **[ACM Console](https://console.aws.amazon.com/acm/home)** → Request a certificate
2. Choose **Request a public certificate**
3. Add:
   - `yourdomain.com`
   - `www.yourdomain.com` (optional)
4. Validation method: **DNS validation** (easiest with Route 53)
5. Confirm and request

🧠 AWS will ask you to create a DNS record in Route 53 — click **“Create record in Route 53”** and AWS will handle it.

✅ Once validated, your certificate status will become **“Issued”**

---

### 🧩 **5. Attach the Certificate to an Application Load Balancer (ALB)**

To use HTTPS, your EC2 traffic needs to go through a **Load Balancer**:

1. Go to **EC2 → Load Balancers** → Create a new **Application Load Balancer**
2. Name: `wordpress-alb`
3. Scheme: Internet-facing
4. Listener: Add **HTTP (port 80)** and **HTTPS (port 443)**
5. Choose your VPC and subnets
6. Under “Security Groups”, allow HTTP and HTTPS
7. Under “Listeners”:
   - Add your **SSL certificate** from ACM
8. Target group: Add your EC2 instance

✅ ALB will handle HTTPS, and forward traffic to EC2 via HTTP.

---

### 🧩 **6. Update Your Route 53 DNS Record to Point to ALB**

1. Go back to Route 53 → Hosted Zone → Click **Create record**
2. Type: **A – Alias**
3. Alias target: Choose your **Load Balancer DNS name**
4. Save the record

✅ Now, `https://yourdomain.com` is secured with SSL, served via ALB.

---

## ⚙️ **BONUS: Automate Everything with CloudFormation or Terraform**

Would you like to:

- ✅ Deploy EC2 + RDS + Security Groups using **CloudFormation YAML/JSON**
- ✅ Automate domain, SSL, and ALB using Infrastructure as Code
- ✅ Set up everything in one command with **Terraform**

