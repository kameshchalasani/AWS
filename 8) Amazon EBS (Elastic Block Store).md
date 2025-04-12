
## 💡 **What is EBS?**

EBS provides **block-level storage** for EC2, just like a virtual hard drive. You can **attach/detach** volumes, choose performance levels, and **persist data even if EC2 is stopped**.

---

## ⚙️ **EBS Volume Types**

Each type is built for a specific use case. Here's a quick overview:

| Type   | Description                                  | Use Case                          | Max IOPS       | Max Throughput |
|--------|----------------------------------------------|------------------------------------|----------------|----------------|
| **gp3**| General Purpose SSD (default, cost-effective) | Boot volumes, dev/test workloads   | 16,000 IOPS    | 1,000 MB/s     |
| **gp2**| Older general-purpose SSD (still used)        | Basic workloads                    | 16,000 IOPS    | 250 MB/s       |
| **io1/io2** | Provisioned IOPS SSD                     | High-performance databases         | 256,000 IOPS   | 4,000 MB/s     |
| **st1**| Throughput Optimized HDD                     | Big data, log processing           | 500 IOPS       | 500 MB/s       |
| **sc1**| Cold HDD                                     | Archival, infrequent access        | 250 IOPS       | 250 MB/s       |

✅ **Use `gp3` for most use cases** — it allows custom IOPS/throughput with better pricing than `gp2`.

---

## 🛠️ **Create & Attach EBS Volume**

---

### 📍 Step 1: Create a Volume

1. Go to **EC2 Console → Elastic Block Store → Volumes**
2. Click **“Create Volume”**
3. Select:
   - **Type** (e.g., `gp3`)
   - **Size** (e.g., 8 GiB)
   - **Availability Zone**: Must match your EC2 instance
4. Click **Create Volume**

---

### 🔗 Step 2: Attach Volume to EC2

1. Select the volume → Click **Actions → Attach volume**
2. Choose your EC2 instance from the dropdown
3. Set device name: e.g., `/dev/xvdf`
4. Click **Attach**

---

### 💻 Step 3: Connect to EC2 and Mount

If you attached a blank volume, SSH into your instance and run:

```bash
# Create a file system (ext4)
sudo mkfs -t ext4 /dev/xvdf

# Create a mount point
sudo mkdir /data

# Mount the volume
sudo mount /dev/xvdf /data

# Optional: Add to /etc/fstab for auto-mount
```

---

## 🔌 Detach Volume (Safely)

1. SSH into EC2 and run:
   ```bash
   sudo umount /data
   ```

2. Go to EC2 → Volumes → Select volume → **Actions → Detach Volume**

Now it's safely detached and can be attached to another instance.

---

## 🧠 Pro Tips

- You can **create snapshots** of volumes to back them up or replicate to other AZs.
- EBS **snapshots are stored in S3**, but not visible in your S3 buckets.
- Use **`io1`/`io2` only for very high IOPS workloads** like databases.


## 🚀 **Option 1: Terraform – Create & Attach EBS Volume**

### 📁 File: `main.tf`

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0c02fb55956c7d316"  # Amazon Linux 2 AMI (update as needed)
  instance_type = "t2.micro"
  key_name      = "your-key-pair"          # Replace with your key pair name

  tags = {
    Name = "EC2WithEBS"
  }
}

resource "aws_ebs_volume" "ebs1" {
  availability_zone = aws_instance.web.availability_zone
  size              = 10
  type              = "gp3"

  tags = {
    Name = "MyExtraVolume"
  }
}

resource "aws_volume_attachment" "ebs_attach" {
  device_name = "/dev/sdf"
  volume_id   = aws_ebs_volume.ebs1.id
  instance_id = aws_instance.web.id
  force_detach = true
}
```

> 💡 Run `terraform init && terraform apply` to provision the EC2 + EBS and attach the volume.

---

## 🚀 **Option 2: CloudFormation – EC2 with Attached EBS**

### 📁 File: `ebs-ec2.yaml`

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: EC2 with EBS Volume

Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-0c02fb55956c7d316  # Amazon Linux 2 (check your region)
      KeyName: your-key-pair          # Replace with your actual key name
      AvailabilityZone: us-east-1a
      BlockDeviceMappings:
        - DeviceName: /dev/xvda
          Ebs:
            VolumeSize: 8
            VolumeType: gp3
        - DeviceName: /dev/sdf
          Ebs:
            VolumeSize: 10
            VolumeType: gp3
      Tags:
        - Key: Name
          Value: EC2WithExtraEBS
```

> 💡 Deploy with:
```bash
aws cloudformation deploy \
  --template-file ebs-ec2.yaml \
  --stack-name EC2WithEBS \
  --capabilities CAPABILITY_NAMED_IAM
```

---

## ✅ Next Steps After Creation

After provisioning:

1. **SSH into EC2**:
   ```bash
   ssh -i your-key.pem ec2-user@<Public-IP>
   ```

2. **Format & mount the EBS volume**:
   ```bash
   sudo mkfs -t ext4 /dev/xvdf
   sudo mkdir /data
   sudo mount /dev/xvdf /data
   ```

3. **(Optional)** Add to `/etc/fstab` for auto-mount on reboot:
   ```bash
   echo "/dev/xvdf /data ext4 defaults,nofail 0 2" | sudo tee -a /etc/fstab
   ```

---
