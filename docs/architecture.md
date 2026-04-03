# Architecture Notes

## Overview

The stack builds a custom VPC with public and private subnets, places an Application Load Balancer in the public tier, and runs an Auto Scaling Group of EC2 instances in the private tier — a standard layered model [editorial]. What ties it together operationally is the CI/CD spine: S3 as the artifact store, CodePipeline orchestrating source-to-deploy, CodeBuild for the build stage, and CodeDeploy handling rolling deploys onto the ASG [from-code]. SSM Parameter Store carries runtime config so EC2 instances never bake secrets into AMIs [editorial], SNS handles deployment event notifications, and EventBridge routes pipeline state changes to downstream consumers [from-code]. CloudWatch with structured log groups and alarms closes the observability loop — meaning the architecture ships with at least a baseline on-call story rather than silent failures [inferred].

## Key Decisions

- CodeDeploy on EC2 with Auto Scaling gives zero-downtime rolling deploys without ECS/EKS overhead, but the deployment lifecycle hooks (BeforeInstall, AfterInstall, ApplicationStart) add 5–15 minutes to deploy time versus a container swap — a real cost for teams doing more than ten deploys per day [inferred].
- Storing artifacts in S3 instead of ECR keeps the pipeline simple but ties you to zip-based deployments; you lose layer caching and immutable image semantics that container-based pipelines provide [editorial].
- SSM Parameter Store standard tier is free up to 10,000 parameters but moves to $0.05 per parameter per month on advanced tier — teams inadvertently hit limits and see cryptic TooManyUpdates errors during deploys [inferred].
- An ALB in front of an ASG means health check misconfiguration (wrong path, wrong threshold) will silently drain all instances during a deploy — there is no built-in CDK guard against this and it is the most common 2am incident for this pattern [editorial].
- CloudWatch log retention defaulting to never-expire is a cost trap — a busy EC2 fleet can accumulate $50–200 per month in CloudWatch Logs storage alone if retention is not explicitly set per log group [inferred].