# 💻 Infrastructure-as-Code Examples
## Terraform & CloudFormation for Cost-Optimized Deployments

---

## 📋 Table of Contents

- [AWS Serverless Stack (Terraform)](#aws-serverless-stack-terraform)
- [Aurora Serverless v2 Configuration](#aurora-serverless-v2-configuration)
- [Lambda Functions (Node.js & Java)](#lambda-functions-nodejs--java)
- [S3 + CloudFront Static Hosting](#s3--cloudfront-static-hosting)
- [GitHub Actions CI/CD Pipeline](#github-actions-cicd-pipeline)
- [Cost Optimization Scripts](#cost-optimization-scripts)

---

## ☁️ AWS Serverless Stack (Terraform)

### Complete Serverless Infrastructure

```hcl
# main.tf - Complete serverless stack for React.js + Node.js + PostgreSQL

terraform {
  required_version = ">= 1.5.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "serverless-stack/terraform.tfstate"
    region = "us-east-1"
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Project     = var.project_name
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}

# Variables
variable "project_name" {
  description = "Project name"
  type        = string
  default     = "my-app"
}

variable "environment" {
  description = "Environment (dev, staging, prod)"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

# S3 Bucket for Frontend
resource "aws_s3_bucket" "frontend" {
  bucket = "${var.project_name}-${var.environment}-frontend"
}

resource "aws_s3_bucket_public_access_block" "frontend" {
  bucket = aws_s3_bucket.frontend.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_versioning" "frontend" {
  bucket = aws_s3_bucket.frontend.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

# CloudFront Distribution
resource "aws_cloudfront_distribution" "frontend" {
  enabled             = true
  is_ipv6_enabled     = true
  default_root_object = "index.html"
  price_class         = "PriceClass_100"  # US, Canada, Europe (cheapest)
  
  origin {
    domain_name = aws_s3_bucket.frontend.bucket_regional_domain_name
    origin_id   = "S3-${aws_s3_bucket.frontend.id}"
    
    s3_origin_config {
      origin_access_identity = aws_cloudfront_origin_access_identity.frontend.cloudfront_access_identity_path
    }
  }
  
  default_cache_behavior {
    allowed_methods  = ["GET", "HEAD", "OPTIONS"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "S3-${aws_s3_bucket.frontend.id}"
    
    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }
    
    viewer_protocol_policy = "redirect-to-https"
    min_ttl                = 0
    default_ttl            = 3600    # 1 hour
    max_ttl                = 86400   # 24 hours
    compress               = true     # Enable gzip/brotli
  }
  
  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }
  
  viewer_certificate {
    cloudfront_default_certificate = true
  }
}

resource "aws_cloudfront_origin_access_identity" "frontend" {
  comment = "OAI for ${var.project_name}-${var.environment}"
}

# Lambda Function (Node.js)
resource "aws_lambda_function" "api" {
  filename         = "lambda.zip"
  function_name    = "${var.project_name}-${var.environment}-api"
  role            = aws_iam_role.lambda.arn
  handler         = "index.handler"
  runtime         = "nodejs18.x"
  memory_size     = 512
  timeout         = 10
  
  environment {
    variables = {
      DB_HOST     = aws_rds_cluster.main.endpoint
      DB_NAME     = aws_rds_cluster.main.database_name
      DB_USER     = aws_rds_cluster.main.master_username
      REDIS_HOST  = aws_elasticache_serverless_cache.main.endpoint[0].address
      ENVIRONMENT = var.environment
    }
  }
  
  vpc_config {
    subnet_ids         = aws_subnet.private[*].id
    security_group_ids = [aws_security_group.lambda.id]
  }
}

# API Gateway HTTP API (cheaper than REST API)
resource "aws_apigatewayv2_api" "main" {
  name          = "${var.project_name}-${var.environment}-api"
  protocol_type = "HTTP"
  
  cors_configuration {
    allow_origins = ["*"]
    allow_methods = ["GET", "POST", "PUT", "DELETE", "OPTIONS"]
    allow_headers = ["*"]
  }
}

resource "aws_apigatewayv2_integration" "lambda" {
  api_id           = aws_apigatewayv2_api.main.id
  integration_type = "AWS_PROXY"
  integration_uri  = aws_lambda_function.api.invoke_arn
}

resource "aws_apigatewayv2_route" "default" {
  api_id    = aws_apigatewayv2_api.main.id
  route_key = "$default"
  target    = "integrations/${aws_apigatewayv2_integration.lambda.id}"
}

resource "aws_apigatewayv2_stage" "default" {
  api_id      = aws_apigatewayv2_api.main.id
  name        = "$default"
  auto_deploy = true
}

# IAM Role for Lambda
resource "aws_iam_role" "lambda" {
  name = "${var.project_name}-${var.environment}-lambda-role"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "lambda.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "lambda_vpc" {
  role       = aws_iam_role.lambda.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaVPCAccessExecutionRole"
}

# Outputs
output "cloudfront_url" {
  value = "https://${aws_cloudfront_distribution.frontend.domain_name}"
}

output "api_endpoint" {
  value = aws_apigatewayv2_api.main.api_endpoint
}

output "database_endpoint" {
  value = aws_rds_cluster.main.endpoint
}
```

---

## 💾 Aurora Serverless v2 Configuration

### Cost-Optimized Database Setup

```hcl
# database.tf - Aurora Serverless v2 with auto-pause

# RDS Cluster (Aurora Serverless v2)
resource "aws_rds_cluster" "main" {
  cluster_identifier      = "${var.project_name}-${var.environment}-db"
  engine                  = "aurora-postgresql"
  engine_mode             = "provisioned"
  engine_version          = "14.6"
  database_name           = var.db_name
  master_username         = var.db_username
  master_password         = var.db_password
  
  # Serverless v2 scaling
  serverlessv2_scaling_configuration {
    min_capacity = var.environment == "prod" ? 0.5 : 0.5
    max_capacity = var.environment == "prod" ? 4.0 : 1.0
  }
  
  # Auto-pause for non-production
  enable_http_endpoint = true
  
  # Backups
  backup_retention_period      = var.environment == "prod" ? 14 : 7
  preferred_backup_window      = "03:00-04:00"
  preferred_maintenance_window = "sun:04:00-sun:05:00"
  
  # Encryption
  storage_encrypted = true
  kms_key_id        = aws_kms_key.rds.arn
  
  # Network
  db_subnet_group_name   = aws_db_subnet_group.main.name
  vpc_security_group_ids = [aws_security_group.rds.id]
  
  # Performance Insights
  enabled_cloudwatch_logs_exports = ["postgresql"]
  
  # Deletion protection
  deletion_protection = var.environment == "prod" ? true : false
  skip_final_snapshot = var.environment != "prod"
  
  tags = {
    Name = "${var.project_name}-${var.environment}-db"
  }
}

resource "aws_rds_cluster_instance" "main" {
  identifier         = "${var.project_name}-${var.environment}-db-instance"
  cluster_identifier = aws_rds_cluster.main.id
  instance_class     = "db.serverless"
  engine             = aws_rds_cluster.main.engine
  engine_version     = aws_rds_cluster.main.engine_version
  
  # Performance Insights
  performance_insights_enabled = true
  performance_insights_retention_period = 7
}

# DB Subnet Group
resource "aws_db_subnet_group" "main" {
  name       = "${var.project_name}-${var.environment}-db-subnet"
  subnet_ids = aws_subnet.private[*].id
  
  tags = {
    Name = "${var.project_name}-${var.environment}-db-subnet"
  }
}

# Security Group for RDS
resource "aws_security_group" "rds" {
  name        = "${var.project_name}-${var.environment}-rds-sg"
  description = "Security group for RDS"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.lambda.id]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# KMS Key for Encryption
resource "aws_kms_key" "rds" {
  description             = "KMS key for RDS encryption"
  deletion_window_in_days = 10
  enable_key_rotation     = true
}

# Variables
variable "db_name" {
  description = "Database name"
  type        = string
  default     = "myapp"
}

variable "db_username" {
  description = "Database username"
  type        = string
  sensitive   = true
}

variable "db_password" {
  description = "Database password"
  type        = string
  sensitive   = true
}

# Outputs
output "db_endpoint" {
  value = aws_rds_cluster.main.endpoint
}

output "db_port" {
  value = aws_rds_cluster.main.port
}
```

---

## ⚡ Lambda Functions (Node.js & Java)

### Node.js Lambda with Connection Pooling

```javascript
// index.js - Node.js Lambda handler

const { Pool } = require('pg');
const Redis = require('ioredis');

// Initialize outside handler (reused across invocations)
const pool = new Pool({
  host: process.env.DB_HOST,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  port: 5432,
  max: 1,  // Lambda: 1 connection per instance
  ssl: { rejectUnauthorized: false }
});

const redis = new Redis({
  host: process.env.REDIS_HOST,
  port: 6379,
  lazyConnect: true,
});

exports.handler = async (event) => {
  console.log('Event:', JSON.stringify(event, null, 2));
  
  try {
    const { httpMethod, path, body } = event;
    
    // Route handling
    if (httpMethod === 'GET' && path === '/users') {
      return await getUsers();
    } else if (httpMethod === 'POST' && path === '/users') {
      return await createUser(JSON.parse(body));
    }
    
    return {
      statusCode: 404,
      body: JSON.stringify({ error: 'Not found' })
    };
    
  } catch (error) {
    console.error('Error:', error);
    return {
      statusCode: 500,
      body: JSON.stringify({ error: 'Internal server error' })
    };
  }
};

async function getUsers() {
  const cacheKey = 'users:all';
  
  // Try cache first
  let users = await redis.get(cacheKey);
  if (users) {
    return {
      statusCode: 200,
      headers: { 'Content-Type': 'application/json' },
      body: users
    };
  }
  
  // Cache miss - query database
  const result = await pool.query('SELECT * FROM users LIMIT 100');
  users = JSON.stringify(result.rows);
  
  // Cache for 5 minutes
  await redis.setex(cacheKey, 300, users);
  
  return {
    statusCode: 200,
    headers: { 'Content-Type': 'application/json' },
    body: users
  };
}

async function createUser(userData) {
  const { name, email } = userData;
  
  const result = await pool.query(
    'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
    [name, email]
  );
  
  // Invalidate cache
  await redis.del('users:all');
  
  return {
    statusCode: 201,
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(result.rows[0])
  };
}
```

### Java Lambda with GraalVM (Fast Cold Starts)

```java
// UserHandler.java - Java Lambda with Spring Boot

package com.example;

import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;
import com.amazonaws.services.lambda.runtime.events.APIGatewayProxyRequestEvent;
import com.amazonaws.services.lambda.runtime.events.APIGatewayProxyResponseEvent;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.jdbc.core.JdbcTemplate;

import java.util.List;
import java.util.Map;

@SpringBootApplication
public class UserHandler implements RequestHandler<APIGatewayProxyRequestEvent, APIGatewayProxyResponseEvent> {

    private static ConfigurableApplicationContext context;
    private static JdbcTemplate jdbcTemplate;
    private static ObjectMapper objectMapper = new ObjectMapper();

    static {
        // Initialize Spring context once (cold start)
        context = SpringApplication.run(UserHandler.class);
        jdbcTemplate = context.getBean(JdbcTemplate.class);
    }

    @Override
    public APIGatewayProxyResponseEvent handleRequest(APIGatewayProxyRequestEvent request, Context context) {
        try {
            String method = request.getHttpMethod();
            String path = request.getPath();

            if ("GET".equals(method) && "/users".equals(path)) {
                return getUsers();
            } else if ("POST".equals(method) && "/users".equals(path)) {
                return createUser(request.getBody());
            }

            return new APIGatewayProxyResponseEvent()
                .withStatusCode(404)
                .withBody("{\"error\": \"Not found\"}");

        } catch (Exception e) {
            context.getLogger().log("Error: " + e.getMessage());
            return new APIGatewayProxyResponseEvent()
                .withStatusCode(500)
                .withBody("{\"error\": \"Internal server error\"}");
        }
    }

    private APIGatewayProxyResponseEvent getUsers() throws Exception {
        List<Map<String, Object>> users = jdbcTemplate.queryForList("SELECT * FROM users LIMIT 100");
        String body = objectMapper.writeValueAsString(users);

        return new APIGatewayProxyResponseEvent()
            .withStatusCode(200)
            .withBody(body);
    }

    private APIGatewayProxyResponseEvent createUser(String requestBody) throws Exception {
        Map<String, String> userData = objectMapper.readValue(requestBody, Map.class);
        String name = userData.get("name");
        String email = userData.get("email");

        jdbcTemplate.update("INSERT INTO users (name, email) VALUES (?, ?)", name, email);

        return new APIGatewayProxyResponseEvent()
            .withStatusCode(201)
            .withBody("{\"message\": \"User created\"}");
    }
}
```

---

## 🌐 S3 + CloudFront Static Hosting

### React.js Deployment Script

```bash
#!/bin/bash
# deploy-frontend.sh - Deploy React app to S3 + CloudFront

set -e

ENVIRONMENT=${1:-dev}
PROJECT_NAME="my-app"
BUCKET_NAME="${PROJECT_NAME}-${ENVIRONMENT}-frontend"
DISTRIBUTION_ID="E1234567890ABC"  # Get from Terraform output

echo "Deploying frontend to ${ENVIRONMENT}..."

# Build React app
echo "Building React app..."
REACT_APP_API_URL="https://api.${ENVIRONMENT}.example.com" npm run build

# Upload to S3
echo "Uploading to S3..."
aws s3 sync build/ s3://${BUCKET_NAME}/ \
  --delete \
  --cache-control "public, max-age=31536000, immutable" \
  --exclude "index.html" \
  --exclude "service-worker.js"

# Upload index.html with no-cache (for SPA routing)
aws s3 cp build/index.html s3://${BUCKET_NAME}/index.html \
  --cache-control "no-cache, no-store, must-revalidate"

# Invalidate CloudFront cache
echo "Invalidating CloudFront cache..."
aws cloudfront create-invalidation \
  --distribution-id ${DISTRIBUTION_ID} \
  --paths "/*"

echo "Deployment complete!"
echo "URL: https://${BUCKET_NAME}.s3-website-us-east-1.amazonaws.com"
```

---

## 🚀 GitHub Actions CI/CD Pipeline

### Complete Deployment Workflow

```yaml
# .github/workflows/deploy.yml

name: Deploy Serverless Stack

on:
  push:
    branches:
      - main
      - develop
  pull_request:
    branches:
      - main

env:
  AWS_REGION: us-east-1
  NODE_VERSION: 18

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Build frontend
        run: npm run build
        env:
          REACT_APP_API_URL: ${{ secrets.API_URL }}
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Deploy frontend to S3
        run: |
          aws s3 sync build/ s3://${{ secrets.S3_BUCKET }}/ --delete
      
      - name: Invalidate CloudFront
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"
      
      - name: Deploy Lambda functions
        run: |
          cd backend
          zip -r lambda.zip .
          aws lambda update-function-code \
            --function-name ${{ secrets.LAMBDA_FUNCTION_NAME }} \
            --zip-file fileb://lambda.zip
      
      - name: Run smoke tests
        run: |
          curl -f https://${{ secrets.APP_URL }}/health || exit 1
```

---

## 💰 Cost Optimization Scripts

### Weekly Cost Report

```bash
#!/bin/bash
# cost-report.sh - Weekly AWS cost analysis

set -e

START_DATE=$(date -d "7 days ago" +%Y-%m-%d)
END_DATE=$(date +%Y-%m-%d)

echo "AWS Cost Report: ${START_DATE} to ${END_DATE}"
echo "================================================"

# Get total cost
TOTAL_COST=$(aws ce get-cost-and-usage \
  --time-period Start=${START_DATE},End=${END_DATE} \
  --granularity DAILY \
  --metrics BlendedCost \
  --query 'ResultsByTime[*].Total.BlendedCost.Amount' \
  --output text | awk '{s+=$1} END {print s}')

echo "Total Cost: \$${TOTAL_COST}"
echo ""

# Get cost by service
echo "Cost by Service:"
aws ce get-cost-and-usage \
  --time-period Start=${START_DATE},End=${END_DATE} \
  --granularity MONTHLY \
  --metrics BlendedCost \
  --group-by Type=DIMENSION,Key=SERVICE \
  --query 'ResultsByTime[0].Groups[*].[Keys[0], Metrics.BlendedCost.Amount]' \
  --output table

echo ""
echo "Top 5 Cost Drivers:"
aws ce get-cost-and-usage \
  --time-period Start=${START_DATE},End=${END_DATE} \
  --granularity MONTHLY \
  --metrics BlendedCost \
  --group-by Type=DIMENSION,Key=SERVICE \
  --query 'sort_by(ResultsByTime[0].Groups, &Metrics.BlendedCost.Amount)[-5:]' \
  --output table
```

### Auto-Pause Enabler

```bash
#!/bin/bash
# enable-auto-pause.sh - Enable auto-pause for all dev/staging databases

set -e

ENVIRONMENTS=("dev" "staging")

for ENV in "${ENVIRONMENTS[@]}"; do
  echo "Enabling auto-pause for ${ENV} databases..."
  
  # List all RDS clusters for environment
  CLUSTERS=$(aws rds describe-db-clusters \
    --query "DBClusters[?contains(DBClusterIdentifier, '${ENV}')].DBClusterIdentifier" \
    --output text)
  
  for CLUSTER in $CLUSTERS; do
    echo "  Modifying ${CLUSTER}..."
    
    aws rds modify-db-cluster \
      --db-cluster-identifier ${CLUSTER} \
      --scaling-configuration AutoPause=true,SecondsUntilAutoPause=300,MinCapacity=0.5,MaxCapacity=1 \
      --apply-immediately
    
    echo "  ✓ Auto-pause enabled for ${CLUSTER}"
  done
done

echo "All ${ENV} databases configured with auto-pause!"
```

---

## 🎯 Key Takeaways

1. **Use Terraform for infrastructure-as-code** (version control, reproducibility)
2. **Enable auto-pause for dev/staging** (76% database cost savings)
3. **Use HTTP API Gateway** (70% cheaper than REST API)
4. **Implement connection pooling** (prevents database exhaustion)
5. **Automate deployments with GitHub Actions** (faster, safer releases)
6. **Monitor costs weekly** (catch anomalies early)

---

## 📞 Next Steps

1. 📊 Use [Cost Calculator](./COST_CALCULATOR.md) to estimate savings
2. 💰 Review [AWS Serverless Cost Comparison](./AWS_SERVERLESS_COST_COMPARISON.md)
3. 🎯 Check [Startup Cost Playbook](./STARTUP_COST_PLAYBOOK.md)
4. 🏗️ Study [Serverless Architecture Reference](./SERVERLESS_ARCHITECTURE_REFERENCE.md)
5. 💾 Read [Database Cost Optimization](./DATABASE_COST_OPTIMIZATION.md)

---

**Last Updated**: 2024-01-15  
**Maintained By**: DevOps Team  
**Version**: 1.0.0
