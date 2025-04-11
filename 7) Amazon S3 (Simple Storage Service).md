

## 🪣 Step-by-Step Guide to S3

---

### 📦 **1. Create a Bucket**

1. Go to **S3 Console**: https://console.aws.amazon.com/s3/
2. Click **“Create bucket”**
3. Fill in:
   - **Bucket name**: must be globally unique (e.g., `my-first-s3-bucket-2025`)
   - **Region**: choose closest to your users
4. **Block all public access** (default is ON – good for private use)
5. Click **Create Bucket**

---

### 🛡️ **2. Configure Permissions**

- Go to the bucket → **Permissions tab**
- To allow public access (optional for static website hosting):
  1. Uncheck **Block all public access**
  2. Add a **bucket policy** like:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "PublicRead",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::your-bucket-name/*"
  }]
}
```

⚠️ Use only if you're intentionally hosting public files.

---

### ⬆️ **3. Upload/Download Files**

- Inside the bucket → Click **“Upload”**
- Add files or folders
- Click **Upload**

**Download:**
- Select any object → Actions → **Download**

**Programmatically (CLI):**

```bash
aws s3 cp myfile.txt s3://your-bucket-name/
aws s3 cp s3://your-bucket-name/myfile.txt .
```

---

### 🌀 **4. Enable Versioning**

Versioning allows you to preserve, retrieve, and restore every version of every object stored in your bucket.

1. Go to bucket → **Properties tab**
2. Under **Bucket Versioning**, click **Edit**
3. Enable versioning → Save changes

Now, every time you upload the same file, S3 stores it as a **new version** (instead of overwriting it).

---

### ♻️ **5. Add Lifecycle Policies**

Lifecycle rules automatically manage files — e.g., transition them to cheaper storage like Glacier or delete them after X days.

1. Go to bucket → **Management tab**
2. Click **“Create lifecycle rule”**
3. Rule name: e.g., `ArchiveOldFiles`
4. Choose:
   - **Prefix** (or apply to all files)
   - Transition to **Glacier/Deep Archive** after 30 days
   - Delete after 365 days (optional)

Click **Create rule**

---

## ✅ Summary

| Task                        | Status   |
|-----------------------------|----------|
| Bucket creation             | ✅        |
| Upload/download             | ✅        |
| Permissions configuration   | ✅        |
| Enable versioning           | ✅        |
| Lifecycle rules             | ✅        |

