## **💰 AWS Billing & Cost Management: Hands-on Guide**  

AWS provides various tools to **monitor and control costs** to avoid unexpected charges. Let’s explore **AWS Free Tier limits, Cost Explorer, and the Billing Dashboard**.  

---

## **1️⃣ Understand AWS Free Tier Limits**  

📍 **Goal:** Learn about **AWS Free Tier** and how to track its usage.  

### **✅ AWS Free Tier Categories**  
AWS Free Tier offers three types of usage:  
1. **Always Free:** Services available free **indefinitely** (e.g., IAM, AWS Lambda free requests).  
2. **12-Months Free:** Free for **one year** after sign-up (e.g., EC2, RDS, S3).  
3. **Trial Offers:** Free for a **limited period** (e.g., AWS Redshift, AWS Glue).  

### **🛠️ Free Tier Limits (Key Services)**  
| **Service**   | **Free Tier Limit** |
|--------------|------------------|
| **EC2**      | 750 hours/month of `t2.micro` |
| **S3**       | 5GB storage |
| **RDS**      | 750 hours/month of `db.t3.micro` |
| **Lambda**   | 1M free requests/month |
| **CloudFront** | 50GB data transfer |

⚠️ **Important:** Exceeding Free Tier limits **incurs charges**. Always monitor your usage!  

---

## **2️⃣ Introduction to AWS Cost Explorer & Billing Dashboard**  

📍 **Goal:** Track usage and avoid unexpected AWS costs.  

### **🛠️ Step 1: Access the AWS Billing Dashboard**  
1️⃣ Sign in to [AWS Console](https://aws.amazon.com/console/).  
2️⃣ Click on your **account name (top-right)** → **Billing Dashboard**.  

### **🛠️ Step 2: Monitor Free Tier Usage**  
1️⃣ Go to **Billing Dashboard → Free Tier Usage Alerts**.  
2️⃣ Review **monthly usage** of Free Tier services.  
3️⃣ Enable **email alerts** when usage is close to limits.  

✅ **Success Check:** You should see a **breakdown of your AWS usage** and **Free Tier limits**.  

---

### **3️⃣ Explore AWS Cost Explorer**  

📍 **Goal:** Analyze and visualize AWS spending trends.  

### **🛠️ Step 1: Open Cost Explorer**  
1️⃣ In the **AWS Console**, go to **Billing Dashboard → Cost Explorer**.  
2️⃣ Click **"Enable Cost Explorer"** (if not enabled).  
3️⃣ View **cost trends by service, region, or date**.  

### **🛠️ Step 2: Create a Budget Alert**  
1️⃣ Go to **Billing Dashboard → Budgets**.  
2️⃣ Click **"Create Budget"** → Select **Cost Budget**.  
3️⃣ Set a **monthly spending limit** (e.g., `$5`).  
4️⃣ Choose **email alerts** for cost overages.  

✅ **Success Check:** You’ll receive **email alerts** if AWS usage **exceeds the budget**. 🎯  

---

## **🔹 Summary**  
✅ **Understand AWS Free Tier limits** to avoid charges.  
✅ **Monitor AWS spending** with the **Billing Dashboard**.  
✅ **Use AWS Cost Explorer** for in-depth cost analysis.  
✅ **Set AWS Budgets & Alerts** to track expenses.  

### **📌 Lab 1: Monitor AWS Free Tier Usage**  
📍 **Goal:** Check AWS Free Tier usage and ensure you stay within limits.  

### **🛠 Steps:**  
1️⃣ **Log in to AWS Console** → Open the **Billing Dashboard**.  
2️⃣ **Click "Free Tier Usage Alerts"** to see how much of the Free Tier you’ve used.  
3️⃣ **Review Usage by Service:** Check **EC2, S3, Lambda, RDS** to avoid overages.  

✅ **Success Check:** You should see your Free Tier usage breakdown.  

---

### **📌 Lab 2: Enable AWS Cost Explorer**  
📍 **Goal:** Analyze AWS cost trends by service, region, and date.  

### **🛠 Steps:**  
1️⃣ **Go to Billing Dashboard** → Click **"Cost Explorer"**.  
2️⃣ **Enable Cost Explorer** (if not already enabled).  
3️⃣ **View Cost Breakdown:** Select different **time ranges** and **services** to see spending.  
4️⃣ **Filter by Service:** Check costs for **EC2, RDS, Lambda, and S3**.  

✅ **Success Check:** You can now visualize AWS costs using graphs.  

---

### **📌 Lab 3: Set a Budget Alert to Prevent Overspending**  
📍 **Goal:** Get email alerts if your AWS bill exceeds a set budget.  

### **🛠 Steps:**  
1️⃣ **Go to Billing Dashboard** → Click **"Budgets"**.  
2️⃣ **Click "Create a Budget"** → Select **Cost Budget**.  
3️⃣ **Enter Budget Amount:** (e.g., `$5` per month).  
4️⃣ **Set Alert Threshold:** Choose **80% of budget** (e.g., `$4`).  
5️⃣ **Enter Email for Alerts** → Click **"Create"**.  

✅ **Success Check:** AWS will **email you if costs exceed 80% of your budget**.  

---

### **📌 Lab 4: Set Up a CloudWatch Billing Alarm**  
📍 **Goal:** Get real-time alerts when AWS costs exceed a certain limit.  

### **🛠 Steps:**  
1️⃣ **Go to AWS CloudWatch Console**.  
2️⃣ Click **"Alarms"** → **Create Alarm**.  
3️⃣ **Select "Billing Metric"** → Choose **EstimatedCharges**.  
4️⃣ **Set Alarm Condition:** Example → Trigger when **costs exceed $5**.  
5️⃣ **Set Notification:** Choose **email/SNS topic** to receive alerts.  
6️⃣ **Click "Create Alarm"**.  

✅ **Success Check:** You will get a **notification when AWS costs go above $5**.  

---

## **🎯 Summary of Hands-on Labs**  
✅ **Check AWS Free Tier Usage** to avoid unexpected charges.  
✅ **Enable AWS Cost Explorer** to analyze spending trends.  
✅ **Set a Budget Alert** to track AWS expenses.  
✅ **Create a CloudWatch Billing Alarm** for real-time cost alerts. 
---

# **🛠 AWS Billing & Cost Management – Hands-on Guide**  

## **📌 Lab 1: Monitor AWS Free Tier Usage**  
📍 **Goal:** Check AWS Free Tier usage and ensure you stay within limits.  

### **🔹 Steps:**  
1️⃣ **Log in to AWS Console** → Open the **Billing Dashboard**:  
   - Go to **[AWS Billing Dashboard](https://console.aws.amazon.com/billing/home#/)**.  

2️⃣ **Click "Free Tier Usage Alerts"** (on the left panel).  

3️⃣ **Check Usage by Service:**  
   - Look for **EC2, S3, Lambda, and RDS usage**.  
   - If any service exceeds limits, AWS **may charge you**.  

✅ **Success Check:** You should see a **breakdown of Free Tier usage** by service.  

---

## **📌 Lab 2: Enable AWS Cost Explorer**  
📍 **Goal:** View AWS costs over time and analyze spending trends.  

### **🔹 Steps:**  
1️⃣ **Go to the Billing Dashboard** → Click **"Cost Explorer"** in the left panel.  

2️⃣ **Enable Cost Explorer** (if not enabled).  
   - Click **"Enable Cost Explorer"** → Wait for activation.  

3️⃣ **View Cost Breakdown:**  
   - **Select a Time Range** (Last 6 months, Last 12 months, etc.).  
   - **Filter by Service** (EC2, S3, Lambda, RDS).  

4️⃣ **View Spending by Region & Time**  
   - Change **Filters** to analyze cost distribution.  

✅ **Success Check:** You can now see **graphs showing AWS costs by service and region**.  

---

## **📌 Lab 3: Set a Budget Alert to Prevent Overspending**  
📍 **Goal:** Get an email alert if AWS spending exceeds a set budget.  

### **🔹 Steps:**  
1️⃣ **Go to AWS Billing Dashboard** → Click **"Budgets"** in the left panel.  

2️⃣ **Click "Create Budget"** → Choose **Cost Budget**.  

3️⃣ **Enter Budget Details:**  
   - **Budget Name:** `AWS-Monthly-Budget`  
   - **Amount:** `$5` per month (or any limit you prefer).  
   - **Time Period:** Monthly  
   - Click **Next**.  

4️⃣ **Set Alert Threshold:**  
   - **Threshold:** **80% of budget** (e.g., `$4`).  
   - **Email Notification:** Enter your email.  

5️⃣ **Review & Create Budget** → Click **"Create"**.  

✅ **Success Check:** AWS will **send an email if spending reaches 80% of $5**.  

---

## **📌 Lab 4: Set Up a CloudWatch Billing Alarm**  
📍 **Goal:** Get real-time alerts when AWS costs exceed a certain amount.  

### **🔹 Steps:**  
1️⃣ **Go to AWS CloudWatch Console** → Open **Alarms** ([CloudWatch Alarms](https://console.aws.amazon.com/cloudwatch/home#alarms:)).  

2️⃣ **Click "Create Alarm"**.  

3️⃣ **Choose "Billing Metrics"** → Select **EstimatedCharges**.  

4️⃣ **Set Condition:**  
   - Choose **Static Threshold** → Set to **$5**.  

5️⃣ **Set Notification:**  
   - **Create SNS Topic** → Enter **your email**.  
   - Click **Confirm Subscription** (AWS sends an email for verification).  

6️⃣ **Review & Create Alarm**.  

✅ **Success Check:** AWS will **send a notification when costs go above $5**.  

---

## **🎯 Summary of Hands-on Labs**  
✅ **Check AWS Free Tier Usage** to avoid unexpected charges.  
✅ **Enable AWS Cost Explorer** to analyze spending trends.  
✅ **Set a Budget Alert** to track AWS expenses.  
✅ **Create a CloudWatch Billing Alarm** for real-time cost alerts.  
