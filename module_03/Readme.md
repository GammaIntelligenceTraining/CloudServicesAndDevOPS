# Module 03 — Compute & Storage (Comprehensive Student Reference)

## 1. Amazon EC2 (Elastic Compute Cloud)
EC2 is your "virtual server" in the cloud. You rent a slice of a physical computer in an AWS data center.

### 1.1 The Two-Meter Billing Model
An EC2 instance is actually two separate things billed differently:
*   **The CPU/RAM Meter:** Bills per-second while the instance is `Running`.
*   **The Disk (EBS) Meter:** Bills per-GB-month as long as the disk exists.
*   **⚠️ CRITICAL GOTCHA:** A `Stopped` instance is **NOT FREE**. You stop paying for the CPU, but the "disk keeps spinning" on your bill because AWS is still storing your data.

### 1.2 The Instance Lifecycle
| State | Compute billing | EBS (Storage) billing | Description |
|---|---|---|---|
| **Pending** | No | **Yes** | Disk is being attached and booted. |
| **Running** | **Yes** | Yes | Fully operational. |
| **Stopped** | No | **Yes** | Like a laptop with the lid closed. |
| **Terminated** | No | No | Deleted forever (if "Delete on Termination" is on). |

### 1.3 EC2 Instance Families (The "T-Shirt" Sizes)
AWS uses a naming convention like `t3.micro`:
*   **Family:** `t` (burstable), `c` (compute), `m` (general), `r` (memory).
*   **Generation:** `3` is newer and often cheaper/faster than `2`.
*   **Size:** micro, small, large, xlarge.

### 1.4 Troubleshooting: "Why can't I SSH?"
1.  **Security Group:** Is port 22 open to your IP address?
2.  **Public IP:** Does your instance have one? (Check the console).
3.  **User Name:** Use `ec2-user` (Amazon Linux), `ubuntu` (Ubuntu), or `admin` (Debian).
4.  **Key Permissions:** Did you `chmod 400 key.pem`?
5.  **VPC/Subnet:** Is there an Internet Gateway attached to the Route Table?

---

## 2. Storage Paradigms: Block vs. Object vs. File

| Dimension | Block (EBS) | Object (S3) | File (EFS) |
| :--- | :--- | :--- | :--- |
| **Analogy** | Laptop Hard Drive | Amazon Warehouse | Office Shared Drive |
| **Unit** | Fixed blocks (4KB) | Objects (Data + Keys) | Files in Folders |
| **Access** | **Exclusive** (1 EC2) | Global (HTTP API) | **Concurrent** (Many EC2s) |
| **Latency** | Sub-millisecond | 100ms+ | Single-digit ms |
| **Best For** | OS Disks, Databases | Backups, Web Assets | Shared Code, Home Dirs |

---

## 3. Amazon S3 (Simple Storage Service)
S3 provides **11 nines of durability** (99.999999999%). Statistically, if you store 10 million objects, you'd lose one every 10,000 years.

### 3.1 S3 Storage Classes & Billing Trap
Picking the wrong class can be 90% more expensive due to **minimum storage durations** and **retrieval fees**.

| Class | Retrieval | Min Duration | Retrieval Fee | Use Case |
|---|---|---|---|---|
| **Standard** | Instant | None | No | Frequent access |
| **Intelligent-Tiering**| Instant | None | No | Changing patterns |
| **Standard-IA** | Instant | 30 Days | **Yes** | Backups (monthly) |
| **Glacier Instant** | Instant | 90 Days | **Yes** | Rare access, need it NOW |
| **Glacier Flexible** | 1min - 12hr | 90 Days | **Yes** | Archives (same-day) |
| **Deep Archive** | 12 - 48 hrs | 180 Days | **Yes** | Regulatory (yearly) |
| **Express One Zone** | **< 10ms** | None | No | ML, High-speed apps |

---

## 4. Linux Command Reference

### 4.1 System & Disk Info
```bash
whoami             # Shows current user
pwd                # Print Working Directory
ls -la             # List all files (including hidden)
df -h              # Show disk space (human-readable)
free -m            # Show RAM usage (in MB)
top                # Real-time process monitor (press 'q' to quit)
```

### 4.2 File Management
```bash
touch file.txt     # Create empty file
mkdir folder       # Create directory
mv old.txt new.txt # Rename or move file
rm -rf folder      # DELETE folder (CAREFUL: no trash bin!)
cat file.txt       # Print file content
nano file.txt      # Text editor
```

### 4.3 Permissions (chmod)
*   `chmod 400 key.pem`  -> Private (Only you read). **Required for SSH**.
*   `chmod 755 script.sh` -> Public (Everyone can run it).

---

## 5. AWS CLI Cheat Sheet (S3)

| Task | Command |
|---|---|
| **List Buckets** | `aws s3 ls` |
| **Upload File** | `aws s3 cp file.txt s3://my-bucket` |
| **Download File**| `aws s3 cp s3://my-bucket/file.txt .` |
| **Format Output**| Append `--output table` or `--output json` |

---

## 6. 🚨 The Mandatory Cleanup Checklist
1.  **Terminate EC2 Instances:** Stops compute charges.
2.  **Delete EBS Volumes:** Stops storage charges.
3.  **Release Elastic IPs:** They cost money if they are "Idle."
4.  **Empty S3 Buckets:** You pay for every GB stored.

---

## 7. Official AWS Resources & Documentation

### Compute (EC2)
*   [Amazon EC2 Documentation](https://docs.aws.amazon.com/ec2/)
*   [EC2 Instance Types Reference](https://aws.amazon.com/ec2/instance-types/)
*   [EBS Billing Guide](https://aws.amazon.com/ebs/pricing/)

### Storage (S3, EBS, EFS)
*   [Amazon S3 Documentation](https://docs.aws.amazon.com/s3/)
*   [S3 Storage Classes Deep Dive](https://aws.amazon.com/s3/storage-classes/)
*   [Amazon EBS (Elastic Block Store)](https://docs.aws.amazon.com/ebs/)
*   [Amazon EFS (Elastic File System)](https://docs.aws.amazon.com/efs/)

### CLI & Tools
*   [AWS CLI Command Reference](https://docs.aws.amazon.com/cli/latest/reference/)
