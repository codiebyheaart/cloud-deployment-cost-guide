# 🏗️ Serverless Architecture Reference
## Complete Guide for Java, Node.js, React.js Applications

---

## 📋 Table of Contents

- [Architecture Overview](#architecture-overview)
- [Service Mappings](#service-mappings)
- [Reference Architectures](#reference-architectures)
- [Implementation Examples](#implementation-examples)
- [Best Practices](#best-practices)
- [Performance Optimization](#performance-optimization)
- [Security Considerations](#security-considerations)

---

## 🎯 Architecture Overview

### Serverless Stack Components

```
┌─────────────────────────────────────────────────────────────────┐
│                         Users / Clients                          │
└────────────────────────────┬────────────────────────────────────┘
                             │
                    ┌────────▼────────┐
                    │   CloudFront    │
                    │   (Global CDN)  │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
   ┌────▼─────┐         ┌────▼─────┐         ┌────▼─────┐
   │    S3    │         │   API    │         │ Lambda@  │
   │ (Static) │         │ Gateway  │         │   Edge   │
   └──────────┘         └────┬─────┘         └──────────┘
                             │
                    ┌────────▼────────┐
                    │  Lambda         │
                    │  Functions      │
                    │  (Node.js/Java) │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
   ┌────▼─────┐         ┌────▼─────┐         ┌────▼─────┐
   │  Aurora  │         │ElastiCache│         │   SQS    │
   │Serverless│         │Serverless │         │  Queue   │
   │ (DB)     │         │  (Cache)  │         │          │
   └──────────┘         └───────────┘         └──────────┘
```

### Key Principles

1. **Pay-Per-Use**: Only pay for actual compute time and requests
2. **Auto-Scaling**: Automatically scale from 0 to millions of requests
3. **No Server Management**: Fully managed by cloud provider
4. **High Availability**: Built-in redundancy across availability zones
5. **Event-Driven**: Triggered by HTTP requests, events, schedules

---

## 🗺️ Service Mappings

### Complete Serverless Stack

| Layer | AWS | GCP | Azure | Purpose |
|-------|-----|-----|-------|----------|
| **Frontend** | S3 + CloudFront | Cloud Storage + CDN | Blob + CDN | Static hosting |
| **API Layer** | API Gateway | API Gateway | API Management | HTTP endpoints |
| **Compute** | Lambda | Cloud Functions / Cloud Run | Functions / Container Apps | Business logic |
| **Database** | Aurora Serverless v2 | Cloud SQL | Flexible Server | Relational data |
| **NoSQL** | DynamoDB | Firestore | Cosmos DB | Document storage |
| **Cache** | ElastiCache Serverless | Memorystore | Cache for Redis | Session/data cache |
| **Queue** | SQS | Pub/Sub | Service Bus | Async processing |
| **Storage** | S3 | Cloud Storage | Blob Storage | Object storage |
| **Auth** | Cognito | Firebase Auth | Azure AD B2C | User authentication |
| **Secrets** | Secrets Manager | Secret Manager | Key Vault | Credentials |
| **Monitoring** | CloudWatch | Cloud Monitoring | Azure Monitor | Observability |

---

## 🏛️ Reference Architectures

### Architecture 1: Simple SaaS Application

**Use Case**: Todo app, CRM, project management  
**Traffic**: 10K-100K requests/month  
**Cost**: $30-80/month

```yaml
Frontend:
  Service: S3 + CloudFront
  Tech: React.js (SPA)
  Build: Vite/Webpack → Static files
  Deployment: GitHub Actions → S3 → CloudFront invalidation
  
Backend:
  Service: Lambda + API Gateway
  Tech: Node.js (Express) or Java (Spring Boot)
  Endpoints:
    - GET /api/users
    - POST /api/users
    - GET /api/tasks
    - POST /api/tasks
  Authentication: JWT tokens (Cognito)
  
Database:
  Service: Aurora Serverless v2 (PostgreSQL)
  Schema:
    - users table
    - tasks table
    - sessions table
  Auto-pause: After 5 minutes idle
  
Caching:
  Service: ElastiCache Serverless (Redis)
  Use: Session storage, API response caching
  
Storage:
  Service: S3
  Use: User uploads, attachments
```

**Architecture Diagram**:

```
User → CloudFront → S3 (React.js)
  ↓
User → CloudFront → API Gateway → Lambda (Node.js)
                                      ↓
                                 ┌────┴────┐
                                 │         │
                            Aurora     ElastiCache
                          Serverless   Serverless
                            (DB)        (Cache)
```

---

### Architecture 2: E-Commerce Platform

**Use Case**: Online store, marketplace  
**Traffic**: 100K-1M requests/month  
**Cost**: $150-350/month

```yaml
Frontend:
  Service: S3 + CloudFront
  Tech: React.js (Next.js SSR)
  Features:
    - Server-side rendering for SEO
    - Image optimization (Lambda@Edge)
    - A/B testing (CloudFront Functions)
  
Backend (Microservices):
  Service: Lambda + API Gateway
  Services:
    - Product Service (Node.js)
    - Order Service (Java)
    - Payment Service (Node.js)
    - Inventory Service (Java)
    - Notification Service (Node.js)
  
Database:
  Service: Aurora Serverless v2 (MySQL)
  Tables:
    - products
    - orders
    - customers
    - inventory
  Read Replicas: 2 (auto-scaling)
  
NoSQL:
  Service: DynamoDB
  Use: Shopping cart, session data
  
Caching:
  Service: ElastiCache Serverless
  Use:
    - Product catalog caching
    - Session storage
    - API response caching
  
Message Queue:
  Service: SQS + SNS
  Use:
    - Order processing (async)
    - Email notifications
    - Inventory updates
  
Payment:
  Service: Lambda + Stripe API
  Security: Secrets Manager for API keys
```

**Architecture Diagram**:

```
User → CloudFront → S3 (Next.js)
  ↓
User → API Gateway → Lambda (Microservices)
                         ↓
                    ┌────┴────┬────────┬────────┐
                    │         │        │        │
                 Aurora   DynamoDB  ElastiCache SQS
               Serverless  (Cart)    (Cache)  (Queue)
                  (DB)
```

---

### Architecture 3: Multi-Tenant SaaS

**Use Case**: B2B SaaS, analytics platform  
**Traffic**: 1M-10M requests/month  
**Cost**: $800-1,500/month

```yaml
Frontend:
  Service: S3 + CloudFront (Multi-region)
  Tech: React.js (SPA)
  Features:
    - Tenant-specific subdomains
    - Custom branding per tenant
    - Real User Monitoring (RUM)
  
Backend:
  Service: Lambda + API Gateway
  Architecture: Multi-tenant with tenant isolation
  Services:
    - Auth Service (tenant validation)
    - Analytics Service (data processing)
    - Reporting Service (PDF generation)
    - Export Service (CSV/Excel)
  
Database:
  Service: Aurora Serverless v2 (PostgreSQL) Multi-AZ
  Strategy: Shared database, tenant_id partitioning
  Tables:
    - tenants
    - users (partitioned by tenant_id)
    - analytics_data (partitioned by tenant_id)
  Performance Insights: Enabled
  
Data Warehouse:
  Service: Redshift Serverless
  Use: Analytics, reporting, BI
  
Caching:
  Service: ElastiCache Serverless (Multi-AZ)
  Strategy: Tenant-specific cache keys
  
Message Queue:
  Service: SQS + SNS
  Use:
    - Background data processing
    - Report generation
    - Email notifications
  
Security:
  Service: WAF + Shield
  Rules:
    - Rate limiting per tenant
    - SQL injection protection
    - XSS protection
```

---

## 💻 Implementation Examples

### Example 1: Node.js Lambda Function

```javascript
// handler.js - Node.js Lambda for API endpoint

const { Pool } = require('pg');
const Redis = require('ioredis');

// Database connection pool (reused across invocations)
const pool = new Pool({
  host: process.env.DB_HOST,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  max: 1, // Lambda: 1 connection per instance
});

// Redis cache (reused across invocations)
const redis = new Redis({
  host: process.env.REDIS_HOST,
  port: 6379,
  lazyConnect: true,
});

exports.handler = async (event) => {
  try {
    const userId = event.pathParameters.userId;
    const cacheKey = `user:${userId}`;
    
    // Try cache first
    let user = await redis.get(cacheKey);
    
    if (user) {
      console.log('Cache hit');
      return {
        statusCode: 200,
        headers: { 'Content-Type': 'application/json' },
        body: user,
      };
    }
    
    // Cache miss - query database
    console.log('Cache miss - querying database');
    const result = await pool.query(
      'SELECT * FROM users WHERE id = $1',
      [userId]
    );
    
    if (result.rows.length === 0) {
      return {
        statusCode: 404,
        body: JSON.stringify({ error: 'User not found' }),
      };
    }
    
    user = result.rows[0];
    
    // Cache for 5 minutes
    await redis.setex(cacheKey, 300, JSON.stringify(user));
    
    return {
      statusCode: 200,
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(user),
    };
    
  } catch (error) {
    console.error('Error:', error);
    return {
      statusCode: 500,
      body: JSON.stringify({ error: 'Internal server error' }),
    };
  }
};
```

**Deployment**:

```yaml
# serverless.yml
service: user-api

provider:
  name: aws
  runtime: nodejs18.x
  memorySize: 512
  timeout: 10
  environment:
    DB_HOST: ${env:DB_HOST}
    DB_NAME: ${env:DB_NAME}
    DB_USER: ${env:DB_USER}
    DB_PASSWORD: ${env:DB_PASSWORD}
    REDIS_HOST: ${env:REDIS_HOST}

functions:
  getUser:
    handler: handler.handler
    events:
      - httpApi:
          path: /users/{userId}
          method: GET
```

---

### Example 2: Java Spring Boot Lambda (GraalVM)

```java
// UserHandler.java - Java Lambda with Spring Boot

package com.example;

import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;
import com.amazonaws.services.lambda.runtime.events.APIGatewayProxyRequestEvent;
import com.amazonaws.services.lambda.runtime.events.APIGatewayProxyResponseEvent;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.jdbc.core.JdbcTemplate;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;

import java.util.Map;

@SpringBootApplication
public class UserHandler implements RequestHandler<APIGatewayProxyRequestEvent, APIGatewayProxyResponseEvent> {

    private static ConfigurableApplicationContext context;
    private static JdbcTemplate jdbcTemplate;
    private static JedisPool jedisPool;

    static {
        // Initialize Spring context once (cold start)
        context = SpringApplication.run(UserHandler.class);
        jdbcTemplate = context.getBean(JdbcTemplate.class);
        jedisPool = new JedisPool(System.getenv("REDIS_HOST"), 6379);
    }

    @Override
    public APIGatewayProxyResponseEvent handleRequest(APIGatewayProxyRequestEvent request, Context context) {
        String userId = request.getPathParameters().get("userId");
        String cacheKey = "user:" + userId;

        try (Jedis jedis = jedisPool.getResource()) {
            // Try cache first
            String cachedUser = jedis.get(cacheKey);
            if (cachedUser != null) {
                return new APIGatewayProxyResponseEvent()
                    .withStatusCode(200)
                    .withBody(cachedUser);
            }

            // Cache miss - query database
            String sql = "SELECT * FROM users WHERE id = ?";
            Map<String, Object> user = jdbcTemplate.queryForMap(sql, userId);

            String userJson = new ObjectMapper().writeValueAsString(user);

            // Cache for 5 minutes
            jedis.setex(cacheKey, 300, userJson);

            return new APIGatewayProxyResponseEvent()
                .withStatusCode(200)
                .withBody(userJson);

        } catch (Exception e) {
            return new APIGatewayProxyResponseEvent()
                .withStatusCode(500)
                .withBody("{\"error\": \"Internal server error\"}");
        }
    }
}
```

**Build with GraalVM Native Image**:

```bash
# Build native image for fast cold starts
./mvnw package -Pnative

# Result: 50ms cold start (vs 5-10s for traditional Java)
```

---

### Example 3: React.js Frontend Deployment

```javascript
// React.js app with API integration

// src/api/client.js
const API_BASE_URL = process.env.REACT_APP_API_URL;

export const getUser = async (userId) => {
  const response = await fetch(`${API_BASE_URL}/users/${userId}`);
  if (!response.ok) {
    throw new Error('Failed to fetch user');
  }
  return response.json();
};

// src/components/UserProfile.jsx
import React, { useEffect, useState } from 'react';
import { getUser } from '../api/client';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    getUser(userId)
      .then(setUser)
      .catch(console.error)
      .finally(() => setLoading(false));
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  if (!user) return <div>User not found</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

export default UserProfile;
```

**Deployment Script**:

```bash
#!/bin/bash
# deploy.sh - Deploy React app to S3 + CloudFront

# Build production bundle
npm run build

# Upload to S3
aws s3 sync build/ s3://my-app-bucket/ --delete

# Invalidate CloudFront cache
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/*"

echo "Deployment complete!"
```

---

## ✅ Best Practices

### 1. Lambda Optimization

```yaml
Memory Sizing:
  - Start with 512MB
  - Monitor CloudWatch metrics
  - Increase if CPU-bound (memory = CPU)
  - Decrease if under-utilized
  - Sweet spot: 1024MB for most apps

Cold Start Reduction:
  - Keep dependencies minimal (<50MB)
  - Use Lambda Layers for shared code
  - Provisioned Concurrency for critical functions
  - GraalVM Native Image for Java (10x faster)

Connection Pooling:
  - Reuse database connections
  - Use RDS Proxy for connection pooling
  - Initialize connections outside handler
  - Set max connections = 1 per Lambda

Timeout Configuration:
  - API endpoints: 10-30 seconds
  - Background jobs: 5-15 minutes
  - Don't set unnecessarily high (costs money)
```

### 2. Database Optimization

```yaml
Aurora Serverless v2:
  - Set minimum ACU to 0.5 (lowest cost)
  - Enable auto-pause for dev/staging (5 min)
  - Use connection pooling (RDS Proxy)
  - Enable Performance Insights
  - Monitor ACU usage and adjust max

Query Optimization:
  - Add indexes for frequently queried columns
  - Use EXPLAIN to analyze slow queries
  - Implement pagination (LIMIT/OFFSET)
  - Cache frequently accessed data

Connection Management:
  - Use RDS Proxy ($15/month)
  - Prevents connection exhaustion
  - Handles 1000s of Lambda connections
  - Automatic failover
```

### 3. Caching Strategy

```yaml
CloudFront (CDN):
  - Cache static assets: 1 year
  - Cache API responses: 5 minutes - 1 hour
  - Use Cache-Control headers
  - Enable compression (gzip/brotli)
  - Target cache hit ratio: >80%

ElastiCache (Redis):
  - Session storage: TTL = session timeout
  - API responses: TTL = 5-60 minutes
  - Database query results: TTL = 1-5 minutes
  - Use cache-aside pattern
  - Implement cache warming for critical data

API Gateway Caching:
  - Enable for frequently accessed endpoints
  - 0.5GB cache: $0.02/hour = $14.60/month
  - Reduces Lambda invocations by 60-80%
  - ROI: Saves $20-40/month on compute
```

### 4. Security Best Practices

```yaml
API Gateway:
  - Enable AWS WAF
  - Implement rate limiting (1000 req/5min per IP)
  - Use API keys for client identification
  - Enable CORS properly
  - Use custom domain with SSL

Lambda:
  - Least privilege IAM roles
  - No hardcoded credentials
  - Use Secrets Manager for sensitive data
  - Enable VPC for database access
  - Encrypt environment variables

Database:
  - Enable encryption at rest (AES-256)
  - Use SSL/TLS for connections
  - Rotate credentials regularly
  - Enable audit logging
  - Restrict network access (Security Groups)
```

---

## ⚡ Performance Optimization

### Response Time Targets

| Metric | Target | Optimization |
|--------|--------|-------------|
| **API Response (p50)** | <100ms | Caching, database indexes |
| **API Response (p95)** | <200ms | Connection pooling, query optimization |
| **API Response (p99)** | <500ms | Provisioned concurrency, read replicas |
| **Frontend Load (p50)** | <1s | CDN, code splitting, lazy loading |
| **Frontend Load (p95)** | <2s | Image optimization, minification |
| **Cold Start (Node.js)** | <500ms | Minimize dependencies |
| **Cold Start (Java)** | <100ms | GraalVM Native Image |

### Optimization Checklist

```yaml
Frontend:
  - [ ] Enable CloudFront compression
  - [ ] Implement code splitting
  - [ ] Lazy load images
  - [ ] Use WebP format for images
  - [ ] Minify CSS/JS
  - [ ] Enable browser caching
  - [ ] Use CDN for static assets

Backend:
  - [ ] Implement caching (Redis)
  - [ ] Use connection pooling
  - [ ] Optimize database queries
  - [ ] Add database indexes
  - [ ] Enable API Gateway caching
  - [ ] Use async processing for heavy tasks
  - [ ] Implement pagination

Database:
  - [ ] Create indexes on frequently queried columns
  - [ ] Use read replicas for read-heavy workloads
  - [ ] Enable query caching
  - [ ] Optimize slow queries (EXPLAIN)
  - [ ] Implement database connection pooling
  - [ ] Use prepared statements
```

---

## 🔒 Security Considerations

### Security Layers

```
┌─────────────────────────────────────────────────────────┐
│  Layer 1: CloudFront + WAF                              │
│  - DDoS protection                                      │
│  - Geo-blocking                                         │
│  - Rate limiting                                        │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│  Layer 2: API Gateway                                   │
│  - API keys                                             │
│  - OAuth/JWT validation                                 │
│  - Request throttling                                   │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│  Layer 3: Lambda                                        │
│  - IAM roles (least privilege)                          │
│  - Input validation                                     │
│  - Business logic authorization                         │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│  Layer 4: Database                                      │
│  - Encryption at rest                                   │
│  - SSL/TLS connections                                  │
│  - Security Groups (network isolation)                  │
└─────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **Serverless = 60-90% cost savings** for low-medium traffic
2. **Auto-pause databases** save 76% for dev/staging
3. **Caching reduces costs** by 50-80% (CloudFront + Redis)
4. **GraalVM Native Image** for Java = 10x faster cold starts
5. **Connection pooling** prevents database exhaustion
6. **Multi-layer security** protects against attacks
7. **Monitor and optimize** weekly for best performance

---

## 📞 Next Steps

1. 📊 Use [Cost Calculator](./COST_CALCULATOR.md) to estimate costs
2. 💰 Review [AWS Serverless Cost Comparison](./AWS_SERVERLESS_COST_COMPARISON.md)
3. 🎯 Check [Startup Cost Playbook](./STARTUP_COST_PLAYBOOK.md)
4. 📚 Read [Database Cost Optimization](./DATABASE_COST_OPTIMIZATION.md)
5. 🌐 Compare [Cloud Providers](./CLOUD_PROVIDER_COMPARISON.md)

---

**Last Updated**: 2024-01-15  
**Maintained By**: Serverless Architecture Team  
**Version**: 1.0.0
