# AWS VPC Enterprise Network Design (Production-Ready)

## 📌 Project Overview

This project demonstrates the design and implementation of a **scalable, secure, and enterprise-grade AWS VPC network architecture**. The goal is to create a shared cloud network platform that can host multiple workloads (admin, web, application, platform, and internal services) and scale for years **without redesign**.

The design strictly follows **CIDR planning best practices**, intentional internet access, and audit-friendly routing.

---

## 🎯 Objectives

* Design a single VPC with long-term scalability
* Use **unequal subnet sizes** based on workload demand
* Implement controlled internet access using route tables
* Enforce network-level isolation (no security groups involved)
* Follow real-world enterprise networking patterns

---

## 🌐 AWS Region

* **Region:** us-east-1 (N. Virginia)

> Note: Availability Zone letters (us-east-1a, 1b, etc.) are account-specific. CIDR logic remains unchanged.

---

## 🧱 VPC Configuration

| Resource  | Value                             |
| --------- | --------------------------------- |
| VPC Name  | Prod-VPC                          |
| VPC CIDR  | 10.0.0.0/16                       |
| Total IPs | 65,536                            |
| Purpose   | Long-term scalable shared network |

---

## 🧮 Subnet Design (Unequal Sizes)

> Allocation strategy: **Largest subnets first**, correct CIDR boundaries, no overlap.

| Subnet Name     | Purpose                  | CIDR         | IP Range                | Internet Access |
| --------------- | ------------------------ | ------------ | ----------------------- | --------------- |
| Shared-Subnet   | Large internal services  | 10.0.0.0/19  | 10.0.0.0 – 10.0.31.255  | ❌ No            |
| Platform-Subnet | Containers / tools       | 10.0.32.0/20 | 10.0.32.0 – 10.0.47.255 | ❌ No            |
| App-Subnet      | Application tier         | 10.0.48.0/21 | 10.0.48.0 – 10.0.55.255 | ❌ No            |
| Web-Subnet      | Web tier                 | 10.0.56.0/22 | 10.0.56.0 – 10.0.59.255 | ❌ No            |
| Edge-Subnet     | Load balancers / ingress | 10.0.60.0/23 | 10.0.60.0 – 10.0.61.255 | ✅ Yes           |
| Admin-Subnet    | Bastion / ops            | 10.0.62.0/24 | 10.0.62.0 – 10.0.62.255 | ✅ Yes           |

---

## 🌍 Internet Gateway (IGW)

| Component   | Configuration |
| ----------- | ------------- |
| IGW Name    | Prod-IGW      |
| Attached To | Prod-VPC      |

The IGW enables **controlled internet access** only for selected subnets.

---

## 🧭 Route Table Architecture

### Public Route Table (Prod-Pub-RT)

Used by Admin and Edge subnets.

| Destination | Target           |
| ----------- | ---------------- |
| 10.0.0.0/16 | Local            |
| 0.0.0.0/0   | Internet Gateway |

---

### Private Route Table (Prod-Pvt-RT)

Used by Web, App, Platform, and Shared subnets.

| Destination | Target |
| ----------- | ------ |
| 10.0.0.0/16 | Local  |

> No default internet route is configured for private subnets.

---

## 🔗 Route Table Associations

| Subnet          | CIDR | IP Range                | Route Table |
| --------------- | ---- | ----------------------- | ----------- |
| Admin-Subnet    | /24  | 10.0.62.0 – 10.0.62.255 | Public-RT   |
| Edge-Subnet     | /23  | 10.0.60.0 – 10.0.61.255 | Public-RT   |
| Web-Subnet      | /22  | 10.0.56.0 – 10.0.59.255 | Private-RT  |
| App-Subnet      | /21  | 10.0.48.0 – 10.0.55.255 | Private-RT  |
| Platform-Subnet | /20  | 10.0.32.0 – 10.0.47.255 | Private-RT  |
| Shared-Subnet   | /19  | 10.0.0.0 – 10.0.31.255  | Private-RT  |

✔ Main route table is **not used**

---

## 🔐 Network Security Behavior (Routing-Based)

* Only **Admin** and **Edge** subnets can access the internet
* Private subnets are fully isolated
* All subnets can communicate internally via local VPC routing

> Security Groups and NACLs are intentionally excluded to keep focus on **pure networking design**.

---

## 🧪 Validation & Testing

### Public Subnet Test (Admin)

```bash
ping google.com
```

✅ Works (internet reachable via IGW)

### Private Subnet Test (Web/App)

```bash
ping google.com
```

❌ Fails (no internet route)

### Internal Communication Test

```bash
ping <private-ip-of-another-subnet>
```

✅ Works (local routing inside VPC)

### Bastion Access Pattern

```text
Laptop → Admin (Public) → Web/App (Private)
```

---

## ⚠️ Failure & Audit Scenarios

### If Internet Gateway is Detached

* All internet access stops
* Internal VPC traffic continues

### If a Private Subnet Uses Public-RT

* Gains unintended internet access
* Security violation
* Audit failure

### Why CIDR Alignment Matters

* CIDR blocks must start on correct binary boundaries
* Incorrect /19 or /20 starting IPs cause overlap or AWS rejection

---

## 🚀 Scalability & Future Growth

* Large unused CIDR space remains in the VPC
* Easy expansion to:

  * Additional AZs
  * New application tiers
  * DR or shared services
* No re-IP or redesign required

---

## 🧠 Key Takeaways

* Designed with enterprise best practices
* Secure by default using routing
* Highly scalable and audit-ready
* Mirrors real-world production architectures

---

## 📎 Author

**Kishan Gollamudi**
DevOps Intern / Cloud Engineer

---

## 🏁 Status

✅ Design Complete
✅ Implemented in AWS Console
✅ Validated & Tested

---


## Screenshots
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7dafaf32-203b-4204-9a65-4130efb2163b" />
### Testing: 
<img width="1920" height="1080" alt="20" src="https://github.com/user-attachments/assets/d63bb63f-6c01-4a38-8b8f-6c46c2e145d5" />
<img width="1920" height="1080" alt="19" src="https://github.com/user-attachments/assets/7fd8eded-750a-4080-b9c7-abc97af73290" />
<img width="1920" height="1080" alt="18" src="https://github.com/user-attachments/assets/13f264af-3495-46ec-8b57-5bce4c07faa4" />
<img width="1920" height="1080" alt="17" src="https://github.com/user-attachments/assets/7e7ad23b-ff8b-4aee-8033-7db485dc1d71" />
<img width="1920" height="1080" alt="16" src="https://github.com/user-attachments/assets/8c5afdc3-ae64-4211-a6ec-a3fb8587cf40" />
<img width="1920" height="1080" alt="15" src="https://github.com/user-attachments/assets/ef5f0ac7-8af1-4b62-a6b7-09ea9ba7db4f" />
<img width="1920" height="1080" alt="14" src="https://github.com/user-attachments/assets/35058dba-24d8-4950-b525-da7ea15a1d7c" />
<img width="1920" height="1080" alt="13" src="https://github.com/user-attachments/assets/e6456313-bd02-4d62-810c-214b9c6036d5" />
<img width="1920" height="1080" alt="12" src="https://github.com/user-attachments/assets/e6e84b69-386f-4f78-98d4-a926bae2bca0" />
<img width="1920" height="1080" alt="11" src="https://github.com/user-attachments/assets/c753f435-980c-4f6d-a237-4cea74e2498e" />
### Internet Gateway: 
<img width="1920" height="1080" alt="10" src="https://github.com/user-attachments/assets/60e23ced-6cda-43b0-a95a-9eeb58236823" />
### Private RT: 
<img width="1920" height="1080" alt="9" src="https://github.com/user-attachments/assets/0f7ff5a2-9ace-4c1a-8c8f-4bdb3cd5a570" />
<img width="1920" height="1080" alt="9 2" src="https://github.com/user-attachments/assets/b9f49e3b-baff-4134-b02b-1073a059a346" />
<img width="1920" height="1080" alt="9 1" src="https://github.com/user-attachments/assets/060b8ff1-f80d-495d-8b7f-77812ac97978" />
### Public RT: 
<img width="1920" height="1080" alt="8" src="https://github.com/user-attachments/assets/41af08ef-8291-4a07-94fd-e0fd6fa060a1" />
<img width="1920" height="1080" alt="8 2" src="https://github.com/user-attachments/assets/1aa38e93-6275-4d5f-a378-6a4cbce89f3f" />
<img width="1920" height="1080" alt="8 1" src="https://github.com/user-attachments/assets/03883f10-4e92-4fe5-9c54-9c935f88ed3f" />
### Admin Subnet
<img width="1920" height="1080" alt="7" src="https://github.com/user-attachments/assets/8a91cd44-2d84-40f3-80bb-6f807b068593" />
### App Subnet
<img width="1920" height="1080" alt="6" src="https://github.com/user-attachments/assets/dc505988-cd37-4d65-ba56-4f3539ebfd3d" />
### Edge Subnet: 
<img width="1920" height="1080" alt="5" src="https://github.com/user-attachments/assets/a2c4bf6a-49ff-4bd8-9aec-eeff70ca7383" />
### Platform Subnet: 
<img width="1920" height="1080" alt="4" src="https://github.com/user-attachments/assets/94508c79-aba9-45a7-ba9e-dcd7e64af0ee" />
### Shared Subnet: 
<img width="1920" height="1080" alt="3" src="https://github.com/user-attachments/assets/1baa01fe-28d2-400c-9340-94ffd15502b9" />
### Web Subnet: 
<img width="1920" height="1080" alt="2" src="https://github.com/user-attachments/assets/b4420125-ea7a-4db9-bf6a-6b50219749c6" />
### VPC: 
<img width="1920" height="1080" alt="1" src="https://github.com/user-attachments/assets/a804b6ca-c817-4ce4-84a3-6cecc9779796" />

