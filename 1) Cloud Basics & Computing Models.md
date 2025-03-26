
# **☁️ Cloud Basics & Computing Models**  

## **🌍 What is Cloud Computing?**  
Cloud computing is the **on-demand delivery** of computing resources (servers, storage, databases, networking, software, and analytics) over the **internet** with a **pay-as-you-go pricing model**.  

💡 **Think of it like this:** Instead of owning a physical server, you rent it from a cloud provider (like AWS, Google Cloud, or Azure) and pay only for what you use.  

### **🖥️ Traditional vs. Cloud Computing**  
| Feature | Traditional IT (On-Premises) | Cloud Computing |
|---------|----------------------|----------------|
| **Infrastructure** | Owned and maintained by a company | Managed by cloud providers |
| **Upfront Cost** | High (Hardware, Maintenance) | Low (Pay-as-you-go) |
| **Scalability** | Manual hardware upgrades | Automatic scaling |
| **Security** | Managed by internal IT teams | Cloud provider ensures compliance and security |
| **Access** | Local | Accessible from anywhere |


---

## **⚡ Cloud Service Models (IaaS, PaaS, SaaS)**  
Cloud services come in **three models**, depending on how much control the user has.  

### **1️⃣ Infrastructure as a Service (IaaS)**  
🔹 **Definition:** Provides virtualized computing resources over the internet.  
🔹 **You manage:** OS, applications, runtime, and networking.  
🔹 **Provider manages:** Hardware, storage, networking, virtualization.  
🔹 **Examples:**  
   - **Amazon EC2** (AWS) – Virtual machines  
   - **Google Compute Engine** – Compute power  
   - **Microsoft Azure Virtual Machines**  

🔹 **📌 Real-World Example:**  
If you were setting up a website, IaaS would be like renting a **virtual machine** where you install your OS and web server.  

---

### **2️⃣ Platform as a Service (PaaS)**  
🔹 **Definition:** Provides a managed environment for developers to build, test, and deploy applications.  
🔹 **You manage:** The application code.  
🔹 **Provider manages:** Infrastructure, OS, runtime, and development tools.  
🔹 **Examples:**  
   - **AWS Elastic Beanstalk** – Deploy web apps easily  
   - **Google App Engine** – Run apps without managing infrastructure  
   - **Microsoft Azure App Services**  

🔹 **📌 Real-World Example:**  
If you want to build a web app, PaaS provides everything needed—just upload your code, and it runs without worrying about servers.  

---

### **3️⃣ Software as a Service (SaaS)**  
🔹 **Definition:** Provides fully functional software applications over the internet.  
🔹 **You manage:** Nothing (just use the service).  
🔹 **Provider manages:** Everything (software, updates, maintenance).  
🔹 **Examples:**  
   - **Google Drive** – Cloud storage  
   - **Microsoft Office 365** – Online productivity tools  
   - **Salesforce** – Customer Relationship Management (CRM)  

🔹 **📌 Real-World Example:**  
Using **Google Docs**—you don’t install any software, just log in and use it. The provider handles updates and storage.  

---

### **📊 Comparison of Cloud Service Models**  
| Feature | **IaaS** | **PaaS** | **SaaS** |
|---------|--------|--------|--------|
| **Management Responsibility** | User manages OS, apps | User manages apps | Fully managed by provider |
| **Flexibility** | High | Medium | Low |
| **Use Case** | Hosting, computing power | App development | End-user applications |
| **Examples** | AWS EC2, Google Compute Engine | AWS Elastic Beanstalk, Azure App Services | Google Docs, Office 365 |

---

## **🚀 Benefits of Cloud Computing**  

✅ **1. Scalability** – Scale up/down resources automatically.  
✅ **2. Cost-Effective** – No upfront costs, only pay for what you use.  
✅ **3. Security** – Providers offer encryption, IAM, and compliance standards.  
✅ **4. High Availability** – Redundant infrastructure ensures uptime.  
✅ **5. Performance** – Global data centers ensure low latency.  
✅ **6. Automatic Updates** – No need to manually install updates.  
✅ **7. Disaster Recovery** – Backup solutions minimize data loss.  

📌 **Example:**  
A startup can launch a website on AWS without buying expensive servers. As traffic grows, AWS automatically scales resources.  

---



### 🚀 **Hands-on Exercises for Cloud Computing Basics**  

These exercises will help you gain **practical experience** with AWS and cloud services.  

---

## **📝 Exercise 1: Set Up an AWS Free Tier Account**  
📌 **Goal:** Create an AWS account and explore the AWS Management Console.  

### **Steps:**  
1. Go to [AWS Free Tier](https://aws.amazon.com/free) and **sign up**.  
2. Enter billing details (AWS may charge $1 for verification, refunded later).  
3. Select **Basic Support (Free)** and complete the sign-up.  
4. Sign in to **AWS Management Console**.  
5. Explore the **AWS services menu** and check the **Billing Dashboard**.  

✅ **Success Check:** You should be able to log in to AWS and see the **AWS Console Dashboard**.  

---

## **🖥️ Exercise 2: Launch a Virtual Machine (IaaS - AWS EC2)**  
📌 **Goal:** Deploy a virtual machine (EC2 instance) in AWS.  

### **Steps:**  
1. Go to **AWS Management Console → EC2**.  
2. Click **Launch Instance**.  
3. Choose **Amazon Linux 2 AMI (Free Tier Eligible)**.  
4. Select **t2.micro (Free Tier Eligible)** instance type.  
5. Configure:  
   - Key Pair: **Create a new key pair** (Download `.pem` file).  
   - Security Group: **Allow SSH (22) and HTTP (80)**.  
6. Click **Launch Instance** and wait until it starts.  
7. **Connect to your instance**:  
   - Open Terminal (Mac/Linux) or PowerShell (Windows).  
   - Run:  
     ```bash
     ssh -i your-key.pem ec2-user@your-ec2-public-ip
     ```
8. Run `uptime` or `ls` to check if it's working.  

✅ **Success Check:** You should be able to **SSH into your EC2 instance** and execute commands.  

---

## **📦 Exercise 3: Store Files in the Cloud (SaaS - AWS S3)**  
📌 **Goal:** Create an S3 bucket and upload a file.  

### **Steps:**  
1. Go to **AWS Management Console → S3**.  
2. Click **Create Bucket** and enter a unique name.  
3. Set the region (e.g., **us-east-1**).  
4. Keep default settings (**Block Public Access** enabled).  
5. Click **Create Bucket**.  
6. Open the bucket and **upload a file** (e.g., an image or text file).  

✅ **Success Check:** Your file is stored in **S3** and can be accessed through the AWS Console.  

---

## **📊 Exercise 4: Deploy a Simple Web App (PaaS - AWS Elastic Beanstalk)**  
📌 **Goal:** Deploy a basic web application using AWS Elastic Beanstalk.  

### **Steps:**  
1. Go to **AWS Elastic Beanstalk**.  
2. Click **Create Application** → Choose **Web Server Environment**.  
3. Select **Platform:** Node.js or Python (or your choice).  
4. Upload a sample application (or use the default one).  
5. Click **Create Environment** and wait for deployment.  
6. Copy the **URL** and open it in a browser.  

✅ **Success Check:** You should see a **deployed web app running on AWS**.  

---

## **🔐 Exercise 5: Secure Your AWS Account (IAM Users & Policies)**  
📌 **Goal:** Create an IAM user with limited permissions.  

### **Steps:**  
1. Go to **AWS Management Console → IAM (Identity & Access Management)**.  
2. Click **Users → Add User**.  
3. Enter a username (e.g., `aws-learner`).  
4. Select **Access Type:** AWS Management Console Access.  
5. Set a strong password.  
6. Click **Next: Permissions** → Attach policy **AmazonS3ReadOnlyAccess**.  
7. Click **Create User** and **log in** with the new IAM user.  

✅ **Success Check:** The new IAM user should be able to **view S3 objects but not delete them**.  
