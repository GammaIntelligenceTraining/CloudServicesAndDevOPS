# Module 04 — Cloud Networking & VPC (Student Handout)

## 1. VPC Core Concepts
A **Virtual Private Cloud (VPC)** is your own private network in the AWS cloud. It is logically isolated from other networks.

*   **Region-bound:** A VPC lives in one specific AWS Region (e.g., `us-east-1`).
*   **Isolation:** No traffic enters or leaves your VPC unless you explicitly configure it.
*   **CIDR Block:** The range of private IP addresses for your VPC (e.g., `10.0.0.0/16`).

---

## 2. CIDR Notation (IP Addressing)
CIDR determines how many IP addresses are in your network.

| Prefix | Total IPs | Usable IPs (AWS) | Use Case |
| :--- | :--- | :--- | :--- |
| `/16` | 65,536 | 65,531 | Typical large VPC size |
| `/24` | 256 | 251 | Typical subnet size |
| `/28` | 16 | 11 | Minimum subnet/VPC size |

> 💡 **The +1 Rule:** Every time you add 1 to the prefix (e.g., /24 to /25), you **halve** the number of IPs.

### The 5 Reserved IPs
In every AWS subnet, 5 IPs are reserved and cannot be used by your instances:
1.  `.0`: Network Address.
2.  `.1`: VPC Router.
3.  `.2`: AWS DNS.
4.  `.3`: Reserved for future use.
5.  `.255`: Network Broadcast (reserved, though AWS doesn't support broadcast).

---

## 3. VPC Components

| Component | Analogy | Description |
| :--- | :--- | :--- |
| **Subnet** | A Floor | A slice of your VPC CIDR bound to **one specific AZ**. |
| **Internet Gateway (IGW)** | The Front Door | Allows two-way communication with the internet. (Free). |
| **Route Table** | Mail Sorter | Rules that tell traffic where to go based on its destination IP. |
| **NAT Gateway** | One-Way Valve | Allows instances in private subnets to reach the internet (outbound only). (**Paid/Expensive!**) |
| **VPC Peering** | A Bridge | A direct connection between two VPCs (even in different accounts or regions). |
| **Transit Gateway** | A Hub | A central hub that connects thousands of VPCs and on-premises networks. |
| **VPC Endpoint** | A Private Tunnel | Connects your VPC to AWS services (like S3) without using the public internet. |

---

## 4. Public vs. Private Subnets
A subnet's "type" is defined purely by its **Route Table**, not its name.

*   **Public Subnet:** Its Route Table has a path to an **Internet Gateway (IGW)** for `0.0.0.0/0`.
*   **Private Subnet:** Its Route Table has NO path to an IGW. It may have a path to a NAT Gateway for outbound-only access.

---

## 5. Security: Security Groups vs. NACLs

| Feature | Security Group (SG) | Network ACL (NACL) |
| :--- | :--- | :--- |
| **Layer** | Instance / ENI level | Subnet level |
| **State** | **Stateful** (Response is auto-allowed) | **Stateless** (Must allow in AND out) |
| **Rules** | Allow only | Allow and Deny |
| **Evaluation** | All rules evaluated | Numbered order (first match wins) |

> 🚀 **Pro Pattern:** Use **Security Group IDs** as a source instead of IP addresses. For example: *"Allow Port 3306 from the Web-Server-SG."*

---

## 6. Troubleshooting Checklist
If you cannot connect to your instance, check these in order:
1.  **State:** Is the instance actually `Running`?
2.  **IGW:** Does the VPC have an Internet Gateway attached?
3.  **Route Table:** Does the subnet have a route to the IGW (`0.0.0.0/0` -> `igw-xxx`)?
4.  **Public IP:** Does your instance have a Public IP address?
5.  **Security Group:** Does the Inbound rule allow your protocol (SSH/HTTP) from your IP?
6.  **NACL:** Does the subnet NACL allow traffic (Remember: Check both Inbound and Outbound!)?
7.  **OS Firewall:** Is the server's internal firewall (ufw/iptables) blocking you?
8.  **Tool:** Use **VPC Reachability Analyzer** in the console for a definitive answer ($0.10/run).

---

## 7. Connectivity Cheat Sheet

### Find your Public IP
```bash
curl https://checkip.amazonaws.com
```

### Test Port Connectivity (Netcat)
```bash
nc -zv <public-ip> 22    # Test if SSH port is open
nc -zv <public-ip> 80    # Test if HTTP port is open
```

### Trace Path (Visual)
Use the **VPC Console -> Reachability Analyzer**. It's the fastest way to find a "hidden" blocking rule.

---

## 8. 🚨 MANDATORY Cleanup Checklist
**Networking resources can be expensive. Do not forget these!**

1.  **Terminate EC2 Instances:** This stops compute costs and releases auto-assigned Public IPs.
2.  **Delete NAT Gateways:** These cost ~$33/month even if they aren't doing anything.
3.  **Release Elastic IPs:** You are charged for every Elastic IP in your account, even if it's attached.
4.  **Delete VPC:** This is the easiest way to clean up everything at once. If it fails, check for running instances or NAT Gateways first.

---

## 9. Official AWS Resources & Documentation

### VPC & IP Addressing
*   [What is Amazon VPC?](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
*   [VPC CIDR blocks](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-cidr-blocks.html)
*   [VPC Route Tables](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)

### Connectivity & Gateways
*   [Internet Gateways](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)
*   [NAT Gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
*   [VPC Peering](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)
*   [Transit Gateway](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html)
*   [VPC Endpoints (PrivateLink)](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html)

### Security & Troubleshooting
*   [Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
*   [Network ACLs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)
*   [VPC Reachability Analyzer](https://docs.aws.amazon.com/vpc/latest/reachability/what-is-reachability-analyzer.html)
