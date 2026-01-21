# 🧮 Cloud Cost Calculator
## Estimate Your Monthly Infrastructure Costs

---

## 📋 Table of Contents

- [How to Use This Calculator](#how-to-use-this-calculator)
- [Traffic-Based Cost Estimation](#traffic-based-cost-estimation)
- [Service-Specific Calculators](#service-specific-calculators)
- [Multi-Environment Cost Projection](#multi-environment-cost-projection)
- [ROI Calculator](#roi-calculator)

---

## 🎯 How to Use This Calculator

### Step 1: Determine Your Traffic Level

Answer these questions:
1. How many active users do you have? ___________
2. How many API requests per month? ___________
3. How much data transfer per month (GB)? ___________
4. What's your database size (GB)? ___________

### Step 2: Find Your Tier

| Users | Requests/Month | Data Transfer | Database Size | Tier |
|-------|----------------|---------------|---------------|------|
| 0-1K | 10K-100K | 10-50GB | 5-10GB | **Starter** |
| 1K-10K | 100K-1M | 50-200GB | 10-50GB | **Growth** |
| 10K-100K | 1M-10M | 200GB-1TB | 50-200GB | **Scale** |
| 100K+ | 10M+ | 1TB+ | 200GB+ | **Enterprise** |

### Step 3: Calculate Costs

Use the tables below based on your tier.

---

## 📊 Traffic-Based Cost Estimation

### Starter Tier (0-1K Users)

#### Input Your Values

| Metric | Your Value | Unit |
|--------|-----------|------|
| **API Requests** | _________ | requests/month |
| **Data Transfer** | _________ | GB/month |
| **Database Size** | _________ | GB |
| **Active Hours/Day** | _________ | hours |

#### Cost Calculation

| Service | Formula | Your Cost |
|---------|---------|----------|
| **Frontend (S3 + CloudFront)** | $5 base + ($0.085 × data_transfer_GB) | $_________ |
| **Backend (Lambda)** | $0.20 × (requests / 1,000,000) + $0.0000166667 × GB-seconds | $_________ |
| **Database (Aurora Serverless)** | $0.12 × 0.5_ACU × active_hours + $0.10 × storage_GB | $_________ |
| **Monitoring (CloudWatch)** | $2-5 fixed | $_________ |
| **DNS (Route 53)** | $1-2 fixed | $_________ |
| **Total** | Sum of above | **$_________** |

#### Example Calculation

```yaml
Input:
  API Requests: 50,000/month
  Data Transfer: 30GB/month
  Database Size: 10GB
  Active Hours: 2 hours/day = 60 hours/month

Calculation:
  Frontend: $5 + ($0.085 × 30) = $7.55
  Backend: $0.20 × (50,000 / 1,000,000) + $0.42 = $0.52
  Database: $0.12 × 0.5 × 60 + $0.10 × 10 = $4.60
  Monitoring: $3
  DNS: $1.50
  
Total: $17.17/month
```

---

### Growth Tier (1K-10K Users)

#### Input Your Values

| Metric | Your Value | Unit |
|--------|-----------|------|
| **API Requests** | _________ | requests/month |
| **Data Transfer** | _________ | GB/month |
| **Database Size** | _________ | GB |
| **Active Hours/Day** | _________ | hours |
| **Cache Enabled?** | Yes / No | - |

#### Cost Calculation

| Service | Formula | Your Cost |
|---------|---------|----------|
| **Frontend (S3 + CloudFront)** | $10 + ($0.085 × data_transfer_GB) | $_________ |
| **Backend (Lambda + API Gateway)** | ($0.20 × requests/1M) + ($0.0000166667 × GB-seconds) + ($3.50 × requests/1M) | $_________ |
| **Database (Aurora Serverless)** | $0.12 × avg_ACU × active_hours + $0.10 × storage_GB | $_________ |
| **Caching (ElastiCache)** | $20-50 (if enabled) | $_________ |
| **Monitoring (CloudWatch + X-Ray)** | $10-25 | $_________ |
| **Security (WAF)** | $10-20 | $_________ |
| **DNS (Route 53)** | $2-5 | $_________ |
| **Total** | Sum of above | **$_________** |

#### Example Calculation

```yaml
Input:
  API Requests: 500,000/month
  Data Transfer: 150GB/month
  Database Size: 40GB
  Active Hours: 8 hours/day = 240 hours/month
  Cache: Yes

Calculation:
  Frontend: $10 + ($0.085 × 150) = $22.75
  Backend: 
    - Lambda: $0.20 × 0.5 = $0.10
    - Compute: $0.0000166667 × 125,000 = $2.08
    - API Gateway: $3.50 × 0.5 = $1.75
    - Total: $3.93
  Database: $0.12 × 1 × 240 + $0.10 × 40 = $32.80
  Caching: $30
  Monitoring: $15
  Security: $15
  DNS: $3
  
Total: $122.48/month
```

---

### Scale Tier (10K-100K Users)

#### Input Your Values

| Metric | Your Value | Unit |
|--------|-----------|------|
| **API Requests** | _________ | requests/month |
| **Data Transfer** | _________ | GB/month |
| **Database Size** | _________ | GB |
| **Multi-AZ Database?** | Yes / No | - |
| **Provisioned Concurrency?** | Yes / No | - |

#### Cost Calculation

| Service | Formula | Your Cost |
|---------|---------|----------|
| **Frontend (S3 + CloudFront + RUM)** | $50 + ($0.085 × data_transfer_GB) | $_________ |
| **Backend (Lambda + API Gateway)** | Complex calculation (see example) | $_________ |
| **Database (Aurora Serverless Multi-AZ)** | $0.12 × avg_ACU × 730 + $0.10 × storage_GB | $_________ |
| **Caching (ElastiCache Multi-AZ)** | $80-150 | $_________ |
| **Message Queue (SQS + SNS)** | $10-30 | $_________ |
| **Monitoring (CloudWatch + X-Ray + RUM)** | $40-80 | $_________ |
| **Security (WAF + Shield)** | $30-60 | $_________ |
| **CI/CD** | $10-30 | $_________ |
| **DNS (Route 53)** | $5-15 | $_________ |
| **Total** | Sum of above | **$_________** |

#### Example Calculation

```yaml
Input:
  API Requests: 5,000,000/month
  Data Transfer: 800GB/month
  Database Size: 150GB
  Multi-AZ: Yes
  Provisioned Concurrency: 5 instances

Calculation:
  Frontend: $50 + ($0.085 × 800) = $118
  Backend:
    - Lambda requests: $0.20 × 5 = $1.00
    - Lambda compute: $0.0000166667 × 1,250,000 = $20.83
    - API Gateway: $3.50 × 5 = $17.50
    - Provisioned Concurrency: 5 × $10.80 = $54.00
    - Total: $93.33
  Database: $0.12 × 4 × 730 + $0.10 × 150 = $365.40
  Caching: $120
  Message Queue: $20
  Monitoring: $60
  Security: $45
  CI/CD: $20
  DNS: $10
  
Total: $851.73/month
```

---

## 🛠️ Service-Specific Calculators

### Lambda Cost Calculator

```yaml
Inputs:
  Requests per month: _________
  Average duration (ms): _________
  Memory allocated (MB): _________

Formula:
  GB-seconds = (requests × duration_seconds × memory_MB) / 1024
  
  Request cost = requests × $0.20 / 1,000,000
  Compute cost = GB-seconds × $0.0000166667
  
  Total = Request cost + Compute cost

Example:
  Requests: 1,000,000
  Duration: 500ms = 0.5s
  Memory: 512MB
  
  GB-seconds = (1,000,000 × 0.5 × 512) / 1024 = 250,000
  
  Request cost = 1,000,000 × $0.20 / 1,000,000 = $0.20
  Compute cost = 250,000 × $0.0000166667 = $4.17
  
  Total = $4.37/month
```

### Aurora Serverless Cost Calculator

```yaml
Inputs:
  Minimum ACU: _________
  Maximum ACU: _________
  Average ACU: _________
  Active hours per month: _________
  Storage (GB): _________
  I/O requests (millions): _________

Formula:
  Compute cost = avg_ACU × active_hours × $0.12
  Storage cost = storage_GB × $0.10
  I/O cost = io_requests_millions × $0.20
  Backup cost = storage_GB × $0.021
  
  Total = Compute + Storage + I/O + Backup

Example (24/7 production):
  Min ACU: 0.5
  Max ACU: 4
  Avg ACU: 1
  Active hours: 730
  Storage: 50GB
  I/O: 5M
  
  Compute = 1 × 730 × $0.12 = $87.60
  Storage = 50 × $0.10 = $5.00
  I/O = 5 × $0.20 = $1.00
  Backup = 50 × $0.021 = $1.05
  
  Total = $94.65/month

Example (auto-pause dev):
  Min ACU: 0.5
  Max ACU: 1
  Avg ACU: 0.5
  Active hours: 60 (2h/day)
  Storage: 20GB
  I/O: 0.5M
  
  Compute = 0.5 × 60 × $0.12 = $3.60
  Storage = 20 × $0.10 = $2.00
  I/O = 0.5 × $0.20 = $0.10
  Backup = 20 × $0.021 = $0.42
  
  Total = $6.12/month (84% savings!)
```

### CloudFront Cost Calculator

```yaml
Inputs:
  Data transfer (GB): _________
  Requests (millions): _________
  Cache hit ratio (%): _________

Formula:
  Data transfer cost = data_GB × $0.085
  Request cost = requests_millions × $0.0075 / 10
  
  Total = Data transfer + Requests

Example:
  Data transfer: 500GB
  Requests: 5M
  Cache hit ratio: 80%
  
  Data transfer cost = 500 × $0.085 = $42.50
  Request cost = 5 × $0.0075 / 10 = $0.00375 × 5M = $3.75
  
  Total = $46.25/month
  
  Note: 80% cache hit ratio means:
  - 80% served from edge (fast, cheap)
  - 20% from origin (slower, more expensive)
  - Origin requests reduced by 80%
```

---

## 🏛️ Multi-Environment Cost Projection

### Complete Stack Cost Estimator

| Environment | Frontend | Backend | Database | Cache | Monitoring | Total |
|-------------|----------|---------|----------|-------|------------|-------|
| **Development** | | | | | | |
| - Requests/month | 10K | 10K | - | - | - | |
| - Active hours | - | - | 60 | - | - | |
| - Estimated cost | $2 | $0.50 | $6 | $0 | $2 | **$10.50** |
| | | | | | | |
| **Staging** | | | | | | |
| - Requests/month | 50K | 50K | - | - | - | |
| - Active hours | - | - | 200 | - | - | |
| - Estimated cost | $5 | $1.50 | $18 | $0 | $3 | **$27.50** |
| | | | | | | |
| **Production** | | | | | | |
| - Requests/month | 500K | 500K | - | - | - | |
| - Active hours | - | - | 730 | - | - | |
| - Estimated cost | $25 | $15 | $95 | $30 | $10 | **$175** |
| | | | | | | |
| **Total (All Environments)** | $32 | $17 | $119 | $30 | $15 | **$213** |

### Your Custom Calculation

| Environment | Frontend | Backend | Database | Cache | Monitoring | Total |
|-------------|----------|---------|----------|-------|------------|-------|
| **Development** | $_____ | $_____ | $_____ | $_____ | $_____ | **$_____** |
| **Staging** | $_____ | $_____ | $_____ | $_____ | $_____ | **$_____** |
| **Production** | $_____ | $_____ | $_____ | $_____ | $_____ | **$_____** |
| **Total** | $_____ | $_____ | $_____ | $_____ | $_____ | **$_____** |

---

## 💰 ROI Calculator

### Traditional vs Serverless Comparison

#### Your Current Architecture (Traditional)

| Service | Quantity | Unit Cost | Monthly Cost |
|---------|----------|-----------|-------------|
| **EC2 Instances** | _____ × | $_____ | $_____ |
| **Load Balancer** | _____ × | $_____ | $_____ |
| **RDS Database** | _____ × | $_____ | $_____ |
| **EBS Storage** | _____ GB × | $_____ | $_____ |
| **Data Transfer** | _____ GB × | $_____ | $_____ |
| **Other** | - | - | $_____ |
| **Total** | | | **$_____** |

#### Proposed Serverless Architecture

| Service | Estimated Cost |
|---------|---------------|
| **S3 + CloudFront** | $_____ |
| **Lambda + API Gateway** | $_____ |
| **Aurora Serverless v2** | $_____ |
| **ElastiCache Serverless** | $_____ |
| **Monitoring** | $_____ |
| **Other** | $_____ |
| **Total** | **$_____** |

#### Savings Calculation

```yaml
Current Cost: $_____/month
Serverless Cost: $_____/month

Monthly Savings: $_____ (___%)
Annual Savings: $_____ × 12 = $_____

Migration Effort: _____ hours
Hourly Rate: $_____/hour
Migration Cost: $_____

ROI Timeline: Migration Cost / Monthly Savings = _____ months
```

#### Example ROI Calculation

```yaml
Current Cost: $450/month (traditional)
Serverless Cost: $180/month (serverless)

Monthly Savings: $270 (60%)
Annual Savings: $270 × 12 = $3,240

Migration Effort: 80 hours
Hourly Rate: $100/hour
Migration Cost: $8,000

ROI Timeline: $8,000 / $270 = 30 months

But considering:
- Reduced operational overhead: $50/month
- Faster deployments: $100/month value
- Better scalability: Priceless

Adjusted ROI: $8,000 / ($270 + $50 + $100) = 19 months
```

---

## 🎯 Cost Optimization Checklist

### Immediate Actions (This Week)

- [ ] Calculate current monthly costs
- [ ] Identify top 3 cost drivers
- [ ] Enable database auto-pause (dev/staging)
- [ ] Set up cost alerts at 50%, 80%, 100%
- [ ] Delete unused resources (old snapshots, idle instances)

### Short-Term Actions (This Month)

- [ ] Migrate frontend to S3 + CloudFront
- [ ] Deploy one API to Lambda (pilot)
- [ ] Implement caching (CloudFront + Redis)
- [ ] Right-size instances (reduce by 1 tier)
- [ ] Enable auto-scaling

### Long-Term Actions (This Quarter)

- [ ] Complete serverless migration
- [ ] Purchase reserved capacity (30-50% savings)
- [ ] Implement storage lifecycle policies
- [ ] Set up cost monitoring dashboard
- [ ] Review costs monthly with team

---

## 📊 Monthly Cost Tracking Template

```yaml
Month: _________

Actual Costs:
  Frontend: $_____
  Backend: $_____
  Database: $_____
  Caching: $_____
  Monitoring: $_____
  Other: $_____
  Total: $_____

Budget: $_____
Variance: $_____ (___%)

Top 3 Cost Drivers:
  1. _________ ($_____)
  2. _________ ($_____)
  3. _________ ($_____)

Optimizations Implemented:
  - _____________________
  - _____________________
  - _____________________

Next Month Actions:
  - _____________________
  - _____________________
  - _____________________
```

---

## 🎯 Key Takeaways

1. **Use this calculator monthly** to track cost trends
2. **Auto-pause saves 76%** for dev/staging databases
3. **Serverless saves 60-90%** for low-medium traffic
4. **Monitor and adjust** based on actual usage
5. **Set alerts** to prevent surprise bills
6. **Review weekly** for the first 3 months
7. **Optimize continuously** - costs should scale with revenue

---

## 📞 Next Steps

1. 📊 Fill out the calculator for your application
2. 💰 Compare with [AWS Serverless Cost Comparison](./AWS_SERVERLESS_COST_COMPARISON.md)
3. 🎯 Choose your tier from [Startup Cost Playbook](./STARTUP_COST_PLAYBOOK.md)
4. 🏗️ Review [Serverless Architecture Reference](./SERVERLESS_ARCHITECTURE_REFERENCE.md)
5. 💾 Read [Database Cost Optimization](./DATABASE_COST_OPTIMIZATION.md)

---

**Last Updated**: 2024-01-15  
**Maintained By**: Cost Optimization Team  
**Version**: 1.0.0

---

**💡 Pro Tip**: Print this calculator and fill it out monthly. Track your progress toward cost optimization goals. Celebrate wins with your team!
