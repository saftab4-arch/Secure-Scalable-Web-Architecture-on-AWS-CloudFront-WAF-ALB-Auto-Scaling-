# 🚀 Secure & Scalable Web Architecture on AWS

### CloudFront + WAF + ALB + Auto Scaling (Multi-AZ)

---

## 📌 Overview

This project is a **continuation of a previous AWS networking and load balancing implementation**, where the initial system consisted of:

* Custom VPC with public and private subnets
* Application Load Balancer (ALB)
* EC2 instances deployed in private subnets

### 🔄 Enhancements in This Phase

To evolve the architecture toward a **production-grade system**, the following components were added:

* 🌐 **Amazon CloudFront** – Global CDN for performance and edge delivery
* 🛡️ **AWS WAF** – Protection against common web exploits and abuse
* ⚖️ **Auto Scaling Group (ASG)** – High availability and self-healing infrastructure

---

## 🎯 Objective

To design and validate a **secure, scalable, and highly available web architecture** that reflects real-world cloud deployment patterns by integrating:

* Edge caching and global distribution
* Layer 7 security controls
* Automatic failover and instance recovery

---

## 🏗️ Architecture Diagram (Logical Flow)

```
                  🌍 Internet Users
                         |
                         v
                +------------------+
                |  CloudFront CDN  |
                +------------------+
                         |
                         v
                +------------------+
                |     AWS WAF      |
                |  (Security Layer)|
                +------------------+
                         |
                         v
                +------------------+
                |  Application LB  |
                |  (Public Subnet) |
                +------------------+
                  /             \
                 v               v
        +---------------+   +---------------+
        | EC2 (Private) |   | EC2 (Private) |
        |   AZ-1        |   |   AZ-2        |
        +---------------+   +---------------+

              Auto Scaling Group
```

---

## 🔄 End-to-End Workflow

1. User sends request to CloudFront distribution
2. CloudFront evaluates cache and forwards request
3. AWS WAF inspects request for malicious patterns
4. Valid requests are forwarded to ALB
5. ALB distributes traffic across EC2 instances
6. EC2 instances (private subnet) process request
7. Response flows back via ALB → CloudFront → User

---

## 🔐 Security Design

* Backend EC2 instances are **not publicly accessible**
* Only ALB is internet-facing
* Security Groups enforce:

  * ALB → EC2 (HTTP only)
* AWS WAF provides:

  * Rate limiting protection
  * Managed OWASP rule sets
* CloudFront acts as an additional **edge security layer**

---

## 🧪 Testing & Validation

### ✅ Load Balancing Validation

* Verified traffic distribution across instances
* Confirmed responses from multiple Availability Zones

---

### ✅ High Availability Test

* Manually terminated one EC2 instance
* Observed:

  * No downtime
  * Traffic continued via second instance
  * Auto Scaling launched replacement instance

---

### ✅ WAF Rate Limiting Test

* Simulated high traffic using repeated curl requests
* Triggered:

  * **HTTP 403 responses from CloudFront**
* Confirmed WAF actively blocking excessive requests

---

### ✅ SQL Injection Simulation

Tested using:

```
?id=1' OR '1'='1
```

Observed behavior:

* Initially:

  * No block (rules in COUNT mode)
* After configuration change:

  * Requests blocked with **403 Forbidden**

---

## ⚠️ HTTP vs HTTPS Design Decision

For this project:

* ALB configured using **HTTP (port 80)**
* HTTPS was intentionally not implemented due to:

  * No custom domain configured
  * SSL (ACM) requires domain ownership

### 🔒 Production Recommendation

* Configure HTTPS listener (443)
* Attach ACM certificate
* Redirect all HTTP → HTTPS traffic

---

## 🐞 Troubleshooting & Lessons Learned

### Issue: Target Group showed no registered instances

* **Cause:** Incorrect ASG / TG linkage
* **Fix:** Reconfigured Auto Scaling attachment

---

### Issue: WAF not blocking malicious requests

* **Cause:** Rules were in COUNT mode
* **Fix:** Switched rules to BLOCK

---

### Issue: Incorrect WAF rule configuration

* **Cause:** Used Geo rule instead of security rule
* **Fix:** Implemented AWS Managed Rule Sets

---

### Issue: Rate limiting not triggering

* **Cause:** Insufficient request volume
* **Fix:** Simulated traffic using scripted curl loop

---

### Issue: CloudFront deletion blocked

* **Cause:** Attached to pricing plan lifecycle
* **Fix:** Disabled distribution and scheduled cancellation

---

### Issue: Residual EBS volume after EC2 termination

* **Cause:** Volume not set to delete on termination
* **Fix:** Manually identified and deleted unused volume

---

## 🧹 Cleanup Process

To avoid unexpected charges, the following resources were removed:

* EC2 instances (terminated)
* Auto Scaling Group
* Application Load Balancer
* Target Groups
* Elastic IPs (released)
* NAT Gateway (deleted)
* EBS volumes (deleted)
* CloudFront distribution (disabled)
* Pricing plan scheduled for cancellation
* WAF left idle (no traffic = no cost impact)

---

## 📚 Key Learnings (Resume Highlights)

* Designed **multi-AZ fault-tolerant architecture**
* Implemented **CloudFront + WAF security layer**
* Built **Auto Scaling with health-based recovery**
* Practiced **real-world failure simulation**
* Performed **security testing (rate limit + injection)**
* Applied **private subnet best practices**
* Understood **AWS billing and cost management**
* Troubleshot real infrastructure issues

---

## 🏁 Conclusion

This project demonstrates a **real-world cloud architecture lifecycle**, evolving from a basic ALB setup into a:

* Scalable
* Secure
* Highly available
* Production-aligned system

It reflects practical experience in **design, testing, troubleshooting, and cost-aware cloud engineering**.
