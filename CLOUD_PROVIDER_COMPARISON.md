# ☁️ Cloud Provider Comparison Guide
## AWS vs GCP vs Azure - Cost Analysis for Java, Node.js, React.js Applications

---

## 📋 Table of Contents

- [Overview](#overview)
- [Service Mapping](#service-mapping)
- [Compute Cost Comparison](#compute-cost-comparison)
- [Database Cost Comparison](#database-cost-comparison)
- [Storage & CDN Comparison](#storage--cdn-comparison)
- [Serverless Comparison](#serverless-comparison)
- [Total Cost Analysis](#total-cost-analysis)
- [Recommendations](#recommendations)

---

## 🎯 Overview

This guide compares **AWS**, **Google Cloud Platform (GCP)**, and **Microsoft Azure** for hosting applications built with **Java, Node.js, React.js, MySQL, and PostgreSQL**.

### Quick Comparison Summary

| Factor | AWS | GCP | Azure |
|--------|-----|-----|-------|
| **Best For** | Serverless, Enterprise | Containers, Data Analytics | .NET, Enterprise, Hybrid |
| **Cost (Low Traffic)** | $37-80/month | $45-95/month | $52-110/month |
| **Cost (Medium Traffic)** | $180-320/month | $210-360/month | $240-400/month |
| **Free Tier** | 12 months | 90 days + Always Free | 12 months |
| **Serverless Maturity** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Ease of Use** | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Documentation** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Community** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

---

## 🗺️ Service Mapping

### Complete Service Equivalents

| Service Category | AWS | GCP | Azure |
|-----------------|-----|-----|-------|
| **Frontend Hosting** | S3 + CloudFront | Cloud Storage + Cloud CDN | Blob Storage + Azure CDN |
| **Serverless Compute** | Lambda | Cloud Functions | Azure Functions |
| **Container Serverless** | Fargate | Cloud Run | Container Apps |
| **Kubernetes** | EKS | GKE | AKS |
| **Virtual Machines** | EC2 | Compute Engine | Virtual Machines |
| **Load Balancer** | ALB/NLB | Cloud Load Balancing | Application Gateway |
| **API Gateway** | API Gateway | API Gateway / Cloud Endpoints | API Management |
| **PostgreSQL (Serverless)** | Aurora Serverless v2 | Cloud SQL | Azure Database Flexible Server |
| **MySQL (Serverless)** | Aurora Serverless v2 | Cloud SQL | Azure Database Flexible Server |
| **Object Storage** | S3 | Cloud Storage | Blob Storage |
| **Cache** | ElastiCache | Memorystore | Azure Cache for Redis |
| **Message Queue** | SQS | Pub/Sub | Service Bus |
| **Monitoring** | CloudWatch | Cloud Monitoring | Azure Monitor |
| **Secrets** | Secrets Manager | Secret Manager | Key Vault |
| **CI/CD** | CodePipeline | Cloud Build | Azure DevOps |
| **DNS** | Route 53 | Cloud DNS | Azure DNS |
| **WAF** | AWS WAF | Cloud Armor | Azure WAF |

---

## 💻 Compute Cost Comparison

### Scenario: Node.js API Backend (100K requests/month)

#### AWS Lambda

```yaml
Configuration:
  Runtime: Node.js 18.x
  Memory: 512MB
  Duration: 500ms average
  Requests: 100,000/month

Cost Breakdown:
  Lambda Requests: 100,000 × $0.20/1M = $0.02
  Lambda Compute: 25,000 GB-seconds × $0.0000166667 = $0.42
  API Gateway: 100,000 × $3.50/1M = $0.35
  
Total: $0.79/month
```

#### GCP Cloud Functions

```yaml
Configuration:
  Runtime: Node.js 18
  Memory: 512MB
  Duration: 500ms average
  Requests: 100,000/month

Cost Breakdown:
  Invocations: 100,000 × $0.40/1M = $0.04
  Compute: 25,000 GB-seconds × $0.0000025 = $0.06
  Networking: 1GB × $0.12/GB = $0.12
  
Total: $0.22/month
```

**Note**: GCP has 2M free invocations/month, making this effectively **$0/month**

#### Azure Functions

```yaml
Configuration:
  Runtime: Node.js 18
  Plan: Consumption
  Memory: 512MB
  Duration: 500ms average
  Requests: 100,000/month

Cost Breakdown:
  Executions: 100,000 × $0.20/1M = $0.02
  Execution Time: 25,000 GB-seconds × $0.000016 = $0.40
  
Total: $0.42/month
```

**Note**: Azure has 1M free executions/month, making this effectively **$0/month**

### Winner: GCP (Free) > Azure (Free) > AWS ($0.79)

---

### Scenario: Java Spring Boot API (100K requests/month)

#### AWS Lambda (GraalVM Native)

```yaml
Configuration:
  Runtime: Java 17 (GraalVM)
  Memory: 1024MB
  Duration: 300ms average
  Requests: 100,000/month

Cost Breakdown:
  Lambda Requests: $0.02
  Lambda Compute: 30,000 GB-seconds × $0.0000166667 = $0.50
  API Gateway: $0.35
  
Total: $0.87/month
```

#### GCP Cloud Run

```yaml
Configuration:
  Container: Java 17 (GraalVM)
  Memory: 1GB
  CPU: 1 vCPU
  Duration: 300ms average
  Requests: 100,000/month

Cost Breakdown:
  Requests: 100,000 × $0.40/1M = $0.04
  CPU: 30,000 vCPU-seconds × $0.00002400 = $0.72
  Memory: 30,000 GB-seconds × $0.00000250 = $0.08
  
Total: $0.84/month
```

#### Azure Container Apps

```yaml
Configuration:
  Container: Java 17 (GraalVM)
  Memory: 1GB
  CPU: 1 vCPU
  Duration: 300ms average
  Requests: 100,000/month

Cost Breakdown:
  vCPU: 30,000 seconds × $0.000024 = $0.72
  Memory: 30,000 GB-seconds × $0.000002 = $0.06
  Requests: Included
  
Total: $0.78/month
```

### Winner: Azure ($0.78) > GCP ($0.84) > AWS ($0.87)

---

### Virtual Machine Comparison (Traditional Hosting)

| Specification | AWS EC2 | GCP Compute Engine | Azure VM |
|---------------|---------|-------------------|----------|
| **2 vCPU, 4GB RAM** | t3.medium: $30.37/month | e2-medium: $24.27/month | B2s: $30.37/month |
| **2 vCPU, 8GB RAM** | t3.large: $60.74/month | e2-standard-2: $48.54/month | B2ms: $60.74/month |
| **4 vCPU, 16GB RAM** | t3.xlarge: $121.47/month | e2-standard-4: $97.09/month | B4ms: $121.47/month |
| **Spot/Preemptible** | 70% discount | 60-91% discount | 70-90% discount |
| **Reserved (1 year)** | 40% discount | 37% discount | 40% discount |
| **Reserved (3 years)** | 60% discount | 55% discount | 60% discount |

### Winner: GCP (15-20% cheaper for general-purpose VMs)

---

## 🗄️ Database Cost Comparison

### PostgreSQL Database (50GB, 8 hours/day active)

#### AWS Aurora Serverless v2

```yaml
Configuration:
  Engine: PostgreSQL 14
  Min ACU: 0.5
  Max ACU: 2
  Active: 240 hours/month
  Auto-pause: Yes (5 min)
  Storage: 50GB

Cost Breakdown:
  Compute: 0.5 ACU × 240h × $0.12 = $14.40
  Storage: 50GB × $0.10 = $5.00
  I/O: 1M requests × $0.20/1M = $0.20
  Backup: 50GB × $0.021 = $1.05
  
Total: $20.65/month
```

#### GCP Cloud SQL (PostgreSQL)

```yaml
Configuration:
  Edition: Enterprise
  Machine: db-f1-micro (shared, 0.6GB)
  Storage: 50GB SSD
  Active: 240 hours/month
  Auto-pause: No (not available)

Cost Breakdown:
  Instance: $0.0150/hour × 730h = $10.95
  Storage: 50GB × $0.17 = $8.50
  Backup: 50GB × $0.08 = $4.00
  
Total: $23.45/month
```

**Note**: GCP doesn't have true serverless PostgreSQL with auto-pause

#### Azure Database for PostgreSQL (Flexible Server)

```yaml
Configuration:
  Tier: Burstable
  Compute: B1ms (1 vCore, 2GB)
  Storage: 32GB
  Active: 730 hours/month (no auto-pause)

Cost Breakdown:
  Compute: $0.0255/hour × 730h = $18.62
  Storage: 32GB × $0.115 = $3.68
  Backup: 32GB × $0.095 = $3.04
  
Total: $25.34/month
```

**Note**: Azure doesn't have auto-pause for PostgreSQL Flexible Server

### Winner: AWS ($20.65) > GCP ($23.45) > Azure ($25.34)

**AWS wins due to auto-pause feature saving 80% during idle time**

---

### MySQL Database (50GB, 8 hours/day active)

#### AWS Aurora Serverless v2 (MySQL)

```yaml
Configuration:
  Engine: MySQL 8.0
  Min ACU: 0.5
  Max ACU: 2
  Active: 240 hours/month
  Auto-pause: Yes
  Storage: 50GB

Cost Breakdown:
  Compute: 0.5 ACU × 240h × $0.12 = $14.40
  Storage: 50GB × $0.10 = $5.00
  I/O: 1M requests × $0.20/1M = $0.20
  Backup: 50GB × $0.021 = $1.05
  
Total: $20.65/month
```

#### GCP Cloud SQL (MySQL)

```yaml
Configuration:
  Edition: Enterprise
  Machine: db-f1-micro (0.6GB)
  Storage: 50GB SSD
  Active: 730 hours/month

Cost Breakdown:
  Instance: $0.0150/hour × 730h = $10.95
  Storage: 50GB × $0.17 = $8.50
  Backup: 50GB × $0.08 = $4.00
  
Total: $23.45/month
```

#### Azure Database for MySQL (Flexible Server)

```yaml
Configuration:
  Tier: Burstable
  Compute: B1ms (1 vCore, 2GB)
  Storage: 32GB
  Active: 730 hours/month

Cost Breakdown:
  Compute: $0.0255/hour × 730h = $18.62
  Storage: 32GB × $0.115 = $3.68
  Backup: 32GB × $0.095 = $3.04
  
Total: $25.34/month
```

### Winner: AWS ($20.65) > GCP ($23.45) > Azure ($25.34)

---

## 📦 Storage & CDN Comparison

### Object Storage (100GB with lifecycle)

#### AWS S3

```yaml
Configuration:
  Standard: 30GB
  Infrequent Access: 40GB
  Glacier: 30GB
  Requests: 100K GET, 10K PUT

Cost Breakdown:
  S3 Standard: 30GB × $0.023 = $0.69
  S3 IA: 40GB × $0.0125 = $0.50
  S3 Glacier: 30GB × $0.004 = $0.12
  Requests: $0.04 + $0.05 = $0.09
  
Total: $1.40/month
```

#### GCP Cloud Storage

```yaml
Configuration:
  Standard: 30GB
  Nearline: 40GB
  Coldline: 30GB
  Requests: 100K GET, 10K PUT

Cost Breakdown:
  Standard: 30GB × $0.020 = $0.60
  Nearline: 40GB × $0.010 = $0.40
  Coldline: 30GB × $0.004 = $0.12
  Requests: $0.04 + $0.05 = $0.09
  
Total: $1.21/month
```

#### Azure Blob Storage

```yaml
Configuration:
  Hot: 30GB
  Cool: 40GB
  Archive: 30GB
  Requests: 100K GET, 10K PUT

Cost Breakdown:
  Hot: 30GB × $0.0184 = $0.55
  Cool: 40GB × $0.01 = $0.40
  Archive: 30GB × $0.002 = $0.06
  Requests: $0.04 + $0.05 = $0.09
  
Total: $1.10/month
```

### Winner: Azure ($1.10) > GCP ($1.21) > AWS ($1.40)

---

### CDN (500GB data transfer, 5M requests)

#### AWS CloudFront

```yaml
Cost Breakdown:
  Data Transfer: 500GB × $0.085 = $42.50
  Requests: 5M × $0.0075/10K = $3.75
  
Total: $46.25/month
```

#### GCP Cloud CDN

```yaml
Cost Breakdown:
  Data Transfer (cache hits): 400GB × $0.08 = $32.00
  Data Transfer (cache misses): 100GB × $0.12 = $12.00
  Requests: 5M × $0.0075/10K = $3.75
  
Total: $47.75/month
```

#### Azure CDN

```yaml
Cost Breakdown:
  Data Transfer: 500GB × $0.087 = $43.50
  Requests: Included
  
Total: $43.50/month
```

### Winner: Azure ($43.50) > AWS ($46.25) > GCP ($47.75)

---

## ⚡ Serverless Comparison

### Complete Serverless Stack (React + Node.js + PostgreSQL)

**Traffic**: 100K requests/month, 10GB database, 100GB CDN transfer

#### AWS Serverless Stack

```yaml
Services:
  - S3 + CloudFront (React.js): $2.50
  - Lambda + API Gateway (Node.js): $0.79
  - Aurora Serverless v2 (PostgreSQL): $20.65
  - CloudWatch: $2.00
  
Total: $25.94/month
```

#### GCP Serverless Stack

```yaml
Services:
  - Cloud Storage + CDN (React.js): $3.20
  - Cloud Functions (Node.js): $0.22
  - Cloud SQL (PostgreSQL): $23.45
  - Cloud Monitoring: $0.00 (free tier)
  
Total: $26.87/month
```

#### Azure Serverless Stack

```yaml
Services:
  - Blob Storage + CDN (React.js): $3.50
  - Azure Functions (Node.js): $0.42
  - PostgreSQL Flexible Server: $25.34
  - Azure Monitor: $2.50
  
Total: $31.76/month
```

### Winner: AWS ($25.94) > GCP ($26.87) > Azure ($31.76)

---

## 💰 Total Cost Analysis

### Scenario 1: MVP/Prototype (10K requests/month)

| Cloud | Frontend | Backend | Database | Monitoring | Total |
|-------|----------|---------|----------|------------|-------|
| **AWS** | $1.50 | $0.40 | $8.60 | $1.50 | **$12.00** |
| **GCP** | $2.00 | $0.00 | $23.45 | $0.00 | **$25.45** |
| **Azure** | $2.50 | $0.00 | $25.34 | $2.00 | **$29.84** |

**Winner: AWS (53% cheaper than GCP, 60% cheaper than Azure)**

---

### Scenario 2: Growing Startup (1M requests/month)

| Cloud | Frontend | Backend | Database | Cache | Monitoring | Total |
|-------|----------|---------|----------|-------|------------|-------|
| **AWS** | $10.00 | $7.85 | $43.20 | $15.00 | $5.00 | **$81.05** |
| **GCP** | $12.00 | $6.50 | $48.50 | $18.00 | $3.00 | **$88.00** |
| **Azure** | $14.00 | $7.20 | $52.00 | $20.00 | $6.00 | **$99.20** |

**Winner: AWS (8% cheaper than GCP, 18% cheaper than Azure)**

---

### Scenario 3: Scale (10M requests/month)

| Cloud | Frontend | Backend | Database | Cache | Monitoring | Total |
|-------|----------|---------|----------|-------|------------|-------|
| **AWS** | $45.00 | $78.50 | $172.80 | $50.00 | $15.00 | **$361.30** |
| **GCP** | $48.00 | $72.00 | $185.00 | $55.00 | $12.00 | **$372.00** |
| **Azure** | $52.00 | $80.00 | $195.00 | $60.00 | $18.00 | **$405.00** |

**Winner: AWS (3% cheaper than GCP, 11% cheaper than Azure)**

---

## 🏆 Recommendations

### Choose AWS If:

✅ You want the **most mature serverless ecosystem**  
✅ You need **Aurora Serverless v2 with auto-pause** (unique feature)  
✅ You're building a **serverless-first application**  
✅ You want the **largest community and best documentation**  
✅ You need **extensive third-party integrations**  
✅ You're optimizing for **cost at low-to-medium traffic**  

**Best For**: Startups, serverless applications, cost-sensitive projects

---

### Choose GCP If:

✅ You prefer **Kubernetes and containers** (GKE is best-in-class)  
✅ You want **simpler pricing and billing**  
✅ You need **big data and analytics** (BigQuery, Dataflow)  
✅ You value **clean UI and developer experience**  
✅ You want **generous free tiers** (Cloud Functions, Cloud Run)  
✅ You're using **TensorFlow or machine learning**  

**Best For**: Container-based apps, data analytics, ML/AI projects

---

### Choose Azure If:

✅ You're building **.NET applications**  
✅ You need **hybrid cloud** (on-premises + cloud)  
✅ You have **Microsoft Enterprise Agreement**  
✅ You need **Active Directory integration**  
✅ You want **Azure DevOps** for CI/CD  
✅ You have **Windows Server or SQL Server licenses**  

**Best For**: Enterprise, .NET apps, hybrid cloud, Microsoft ecosystem

---

## 📊 Cost Efficiency Summary

### By Traffic Level

| Traffic Level | Best Choice | Reason |
|---------------|-------------|--------|
| **MVP (0-100K req/month)** | AWS | Auto-pause saves 76%, best free tier for serverless |
| **Growth (100K-5M req/month)** | AWS | Serverless pricing advantage, mature ecosystem |
| **Scale (5M-50M req/month)** | AWS/GCP | Similar costs, choose based on features |
| **Enterprise (50M+ req/month)** | GCP | Better VM pricing, Kubernetes excellence |

### By Workload Type

| Workload | Best Choice | Reason |
|----------|-------------|--------|
| **Serverless APIs** | AWS | Lambda + Aurora Serverless v2 unmatched |
| **Containerized Apps** | GCP | Cloud Run + GKE best-in-class |
| **Static Websites** | AWS | S3 + CloudFront most cost-effective |
| **Data Analytics** | GCP | BigQuery, Dataflow superior |
| **.NET Applications** | Azure | Native .NET support, best tooling |
| **Machine Learning** | GCP | TensorFlow, Vertex AI leadership |
| **Hybrid Cloud** | Azure | Azure Arc, best on-prem integration |

---

## 💡 Cost Optimization Tips

### Multi-Cloud Strategy

```yaml
Recommended Approach:
  Frontend: AWS S3 + CloudFront (cheapest, most reliable)
  Backend: AWS Lambda (best serverless pricing)
  Database: AWS Aurora Serverless v2 (auto-pause feature)
  Analytics: GCP BigQuery (pay-per-query, no infrastructure)
  ML/AI: GCP Vertex AI (best ML platform)
  CI/CD: GitHub Actions (free for public repos)
  
Result: 30-50% cost savings vs single-cloud
```

### Free Tier Maximization

```yaml
AWS (12 months):
  - Lambda: 1M requests/month
  - S3: 5GB storage
  - RDS: 750 hours/month (db.t2.micro)
  - CloudFront: 50GB transfer
  
GCP (Always Free):
  - Cloud Functions: 2M invocations/month
  - Cloud Run: 2M requests/month
  - Cloud Storage: 5GB
  - BigQuery: 1TB queries/month
  
Azure (12 months):
  - Functions: 1M executions/month
  - Blob Storage: 5GB
  - SQL Database: 250GB
  - App Service: 10 web apps
```

---

## 🎯 Key Takeaways

1. **AWS is 10-20% cheaper** for serverless workloads (low-medium traffic)
2. **GCP is 15-20% cheaper** for VM-based workloads
3. **Azure is best for .NET** and enterprise/hybrid scenarios
4. **Aurora Serverless v2 auto-pause** is AWS's killer feature (saves 76%)
5. **GCP has the best free tier** for serverless (Cloud Functions, Cloud Run)
6. **Multi-cloud can save 30-50%** by using best-of-breed services
7. **Start with AWS for MVP**, evaluate GCP/Azure at scale

---

## 📞 Next Steps

1. 📊 Use [Cost Calculator](./COST_CALCULATOR.md) to estimate your specific costs
2. 🏗️ Review [Serverless Architecture Reference](./SERVERLESS_ARCHITECTURE_REFERENCE.md)
3. 💰 Read [AWS Serverless Cost Comparison](./AWS_SERVERLESS_COST_COMPARISON.md)
4. 📚 Check [Database Cost Optimization](./DATABASE_COST_OPTIMIZATION.md)
5. 🎯 See [Startup Cost Playbook](./STARTUP_COST_PLAYBOOK.md)

---

**Last Updated**: 2024-01-15  
**Maintained By**: Multi-Cloud Architecture Team  
**Version**: 1.0.0

---

**💡 Pro Tip**: Use AWS for your MVP to maximize cost savings with serverless + auto-pause. Evaluate GCP/Azure only when you reach 10M+ requests/month or have specific workload requirements (containers, .NET, ML/AI).
