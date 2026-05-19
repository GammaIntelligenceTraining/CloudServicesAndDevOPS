# Module 05 — Security & Observability (Student Handout)

## 1. The Shared Responsibility Model
The core rule of AWS: **"AWS secures the cloud, YOU secure what you put in it."**

*   **AWS's Job (Security OF the Cloud):** Physical security, hardware, fiber cables, and the software that runs the virtualization (The Hypervisor/Nitro System).
*   **Your Job (Security IN the Cloud):** Patching your Linux/Windows OS, setting firewall rules (SGs/NACLs), writing IAM policies, and encrypting your data.

---

## 2. IAM (Identity and Access Management)

### 2.1 The Principle of Least Privilege
Always give a user or machine the **minimum** permissions they need. 
*   *Bad:* Giving a developer "AdministratorAccess" to fix one S3 bucket.
*   *Good:* Giving that developer "S3ReadOnly" access to only the specific bucket they need.

### 2.2 IAM Policy Anatomy (JSON)
IAM is a language. Every policy is a "sentence" built with these parts:
*   **Effect:** `Allow` or `Deny`. (**Note:** Deny always wins!)
*   **Action:** What you can do (e.g., `s3:ListBucket`, `ec2:RunInstances`).
*   **Resource:** What you can do it to (ARN - Amazon Resource Name).
*   **Condition:** Extra rules (e.g., "Only from this IP" or "Only if using MFA").

### 2.3 IAM Roles (The "Temporary Hat")
Roles are for machines (EC2, Lambda) or users who need temporary access. 
*   **No Passwords:** Roles provide temporary security tokens that rotate automatically.
*   **Safety:** Never store permanent "Access Keys" inside your application code. Use a Role instead.

| Common Role | Typical Permissions | Used By |
| :--- | :--- | :--- |
| **EC2 S3 Reader** | `s3:GetObject`, `s3:ListBucket` | EC2 Instances |
| **Lambda Execution** | `logs:CreateLogGroup`, `logs:PutLogEvents` | Lambda Functions |
| **SSM Core** | `ssm:GetParameter`, `ssm:UpdateInstanceInformation` | EC2 (for Systems Manager) |

---

## 3. Secrets & Encryption

### 3.1 AWS KMS (Key Management Service)
KMS is your cloud's "Locksmith." It manages the **Master Keys** used to encrypt your disks, databases, and S3 buckets.
*   **Centralized Control:** You use KMS to define who can use which keys through **Key Policies**.
*   **Hardware Security (HSM):** Your keys are stored in physical, tamper-proof hardware that even AWS employees cannot access.
*   **Envelope Encryption (Why it's fast):** 
    *   KMS gives you a **Data Key** (the small key).
    *   You encrypt your big file *locally* with the Data Key.
    *   The **Master Key** (the big key) stays safe inside KMS and is only used to encrypt the small Data Key.
    *   *Result:* You can encrypt petabytes of data without sending it all over the network to KMS.
*   **Deletion Safety:** You cannot delete a KMS key instantly. There is a **7 to 30 day** mandatory waiting period to prevent accidental data loss.

### 3.2 SSM Parameter Store vs. Secrets Manager
*   **Parameter Store:** Great for non-sensitive settings (URLs) or basic secrets. It is generally **free** for standard parameters.
*   **Secrets Manager:** Built for highly sensitive data (DB passwords). It can **automatically rotate** (change) your passwords every 30 days without you doing anything.

---

## 4. Observability (The "Eyes" of the Cloud)

### 4.1 The Three Pillars
*   **Metrics (Numbers):** "My CPU is at 45%." (CloudWatch Metrics)
*   **Logs (Events):** "Error: Connection timed out." (CloudWatch Logs)
*   **Traces (The Journey):** Tracking a request through multiple services. (AWS X-Ray)

**Goal:** Reduce **MTTR (Mean Time To Resolution)**—the time it takes to fix a problem once it's detected.

### 4.2 CloudWatch Deep Dive
*   **Namespaces:** Containers for metrics (e.g., `AWS/EC2`).
*   **Dimensions:** Labels that identify a metric (e.g., `InstanceId=i-12345`).
*   **Standard Metrics:** Free and automatic (CPU, Disk, Network).
*   **Custom Metrics:** You push these yourself (e.g., **Memory Usage**, which EC2 doesn't track by default).
*   **⚠️ High Cardinality:** Do not use unique values like "User IP" as a Dimension. It creates thousands of unique metrics and will drastically increase your bill.

### 4.3 CloudWatch vs. CloudTrail
| Feature | CloudWatch | CloudTrail |
| :--- | :--- | :--- |
| **Purpose** | Performance & Health | Auditing & API History |
| **Analogy** | The Speedometer/Dashboard | The Dashcam/Black Box |
| **Question** | "Is my app healthy?" | "Who deleted this server?" |
| **Components** | Metrics, Logs, Alarms | Trails, Event History |

### 4.4 EventBridge (The Nervous System)
Automates your cloud using "If-Then" logic.
*   *Example:* "**If** an EC2 instance stops, **Then** trigger a Lambda to send an alert."

---

## 5. Troubleshooting & Tips
*   **Access Denied?** Check your IAM JSON. Remember that `arn:aws:s3:::bucket` (the bucket) is different from `arn:aws:s3:::bucket/*` (the files inside).
*   **Alarm not firing?** CloudWatch standard monitoring is **5-minute intervals**. It might take a few minutes for the "red" state to show up.
*   **Future Note:** As of May 31, 2026, **CloudTrail Lake** is being phased out. Use **CloudWatch Logs Insights** for your long-term audit searching needs.

---

## 6. Lab Commands Cheat Sheet

### Install Stress Tool (Amazon Linux 2023)
```bash
sudo dnf install stress -y
```

### Stress the CPU
```bash
# Run 1 CPU worker for 5 minutes
stress --cpu 1 --timeout 300
```

### Get a Secret via CLI
```bash
aws ssm get-parameter --name /prod/db/password --with-decryption --region <YOUR-REGION>
```

---

## 7. Self-Check Questions
1.  Which service would you use to find out **who** deleted an S3 bucket yesterday?
2.  Why is it safer to use an **IAM Role** for an EC2 instance than an Access Key?
3.  What is the benefit of **Secrets Manager** over **Parameter Store** for a database password?
4.  If a CloudWatch Alarm is set to "Greater than 40" and your CPU is 41, will it turn red instantly?
5.  What does **Envelope Encryption** protect? (The Data, the Master Key, or the Data Key?)

---

## 8. Global Multi-Cloud Reference (AWS vs. Azure vs. GCP)

| Category | Service Type | **AWS** | **Microsoft Azure** | **Google Cloud (GCP)** |
| :--- | :--- | :--- | :--- | :--- |
| **Compute** | Virtual Machines | **EC2** | Azure VMs | Compute Engine (GCE) |
| **Containers** | Kubernetes | **EKS** | AKS | GKE |
| **Storage** | Object Storage | **S3** | Blob Storage | Cloud Storage (GCS) |
| **Networking**| Virtual Network | **VPC** | Virtual Network (VNet) | VPC (Global) |
| **Security** | Identity (IAM) | **IAM** | **Microsoft Entra ID** | Cloud IAM |
| **Observability**| Unified Suite | **CloudWatch** | **Azure Monitor** | **Google Cloud Ops** |
| **Auditing** | CloudTrail | Activity Log | Cloud Audit Logs |

*   **Azure Tip:** Look for the word **"Defender"** for security and **"Entra"** for identity.
*   **GCP Tip:** Google focuses on **AI-driven** operations and high-performance data tools like **BigQuery**.
*   **Concepts > Names:** Don't memorize the names. Master the **concepts**.

---

## 9. Official AWS Resources & Documentation

### Foundational Security
*   [Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/)
*   [IAM Documentation](https://docs.aws.amazon.com/iam/)
*   [Security Best Practices in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

### Secrets & Encryption
*   [AWS KMS Documentation](https://docs.aws.amazon.com/kms/)
*   [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/)
*   [SSM Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)

### Observability & Monitoring
*   [Amazon CloudWatch](https://docs.aws.amazon.com/cloudwatch/)
*   [AWS CloudTrail](https://docs.aws.amazon.com/cloudtrail/)
*   [Amazon EventBridge](https://docs.aws.amazon.com/eventbridge/)
*   [AWS X-Ray (Tracing)](https://docs.aws.amazon.com/xray/)
