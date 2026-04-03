# Cost Model

## Overview

This is a reference cost model. Actual costs vary by usage, region, and configuration.

## Key Cost Drivers

- CodeDeploy on EC2 with Auto Scaling gives zero-downtime rolling deploys without ECS/EKS overhead, but the deployment lifecycle hooks (BeforeInstall, AfterInstall, ApplicationStart) add 5–15 minutes to deploy time versus a container swap — a real cost for teams doing more than ten deploys per day [inferred].
- SSM Parameter Store standard tier is free up to 10,000 parameters but moves to $0.05 per parameter per month on advanced tier — teams inadvertently hit limits and see cryptic TooManyUpdates errors during deploys [inferred].
- CloudWatch log retention defaulting to never-expire is a cost trap — a busy EC2 fleet can accumulate $50–200 per month in CloudWatch Logs storage alone if retention is not explicitly set per log group [inferred].

## Estimated Monthly Cost

| Component | Dev (₹) | Staging (₹) | Production (₹) |
|-----------|---------|-------------|-----------------|
| Compute   | ₹2,000–5,000 | ₹8,000–15,000 | ₹25,000–60,000 |
| Database  | ₹1,500–3,000 | ₹5,000–12,000 | ₹15,000–40,000 |
| Networking| ₹500–1,000   | ₹2,000–5,000  | ₹5,000–15,000  |
| Monitoring| ₹200–500     | ₹1,000–2,000  | ₹3,000–8,000   |
| **Total** | **₹4,200–9,500** | **₹16,000–34,000** | **₹48,000–1,23,000** |

> Estimates based on ap-south-1 (Mumbai) pricing. Actual costs depend on traffic, data volume, and reserved capacity.

## Cost Optimization Strategies

- Use Savings Plans or Reserved Instances for predictable workloads
- Enable auto-scaling with conservative scale-in policies
- Use DynamoDB on-demand for dev, provisioned for production
- Leverage S3 Intelligent-Tiering for infrequently accessed data
- Review Cost Explorer weekly for anomalies
