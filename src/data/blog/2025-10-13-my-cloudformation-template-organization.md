---
title: "My CloudFormation Template Organization"
pubDatetime: 2025-10-13
description: "A structured approach to organizing AWS CloudFormation templates for containerized web applications, covering VPC networking, ECS Fargate, RDS, and automated deployment processes."
slug: my-cloudformation-template-organization
tags:
  - aws
  - infrastructure
  - containers
draft: false
---

I needed a repeatable way to deploy web apps on AWS. I developed a baseline structure that works for containerized web apps with a database. Then I modify and tweak it based on specific project requirements.

This setup handles VPC networking, ECS Fargate containers, RDS database, and load balancing. It includes ECS auto-scaling configuration and has separate test and prod environments. For my example project, it hosts API, Migrator, and Worker services.

## My Template Structure

```plaintext
infrastructure/
├── cloudformation/
│   ├── parameters/
│   │   ├── test.json
│   │   └── prod.json
│   ├── 01-base-vpc-networking.json
│   ├── 02-base-security-groups.json
│   ├── 03-data-rds.json
│   ├── 04-compute-ecs-cluster.json
│   ├── 05-compute-alb.json
│   └── 06-compute-fargate-scaling.json
├── ecs/
│   ├── api-task-definition.json
│   ├── worker-task-definition.json
│   └── migrator-task-definition.json
├── scripts/
│   ├── deploy-environment.sh
│   ├── deploy-scaling.sh
│   ├── setup-secrets.sh
│   └── resolve-secrets.sh
└── README.md
```

Some notes on the structure:

- **Traffic flow:** Internet → ALB (public subnets) → ECS tasks (private subnets) → RDS (private subnets). ECS pulls images from ECR via VPC endpoint. NAT Gateway handles outbound traffic from private subnets.
- **Numbering enforces deployment order.** CloudFormation stacks have dependencies. You can't create an ECS service before the VPC exists. The numbered prefix makes deployment sequence explicit.
- **parameters/** contains environment-specific values, like CIDR blocks, instance sizes, database credentials, anything that differs between test and prod.
- **ecs/** contains task definitions for each service (for my project example: API, Worker, Migrator). These define container specs, CPU, memory, env variables, and secrets.
- **scripts/** automates deployment. `deploy-environment.sh` deploys stacks 01-05 in sequence. `setup-secrets.sh` creates secrets in AWS Secrets Manager during initial setup. `resolve-secrets.sh` replaces secret placeholders with actual ARNs before every deployment. `deploy-scaling.sh` deploys auto-scaling after services run.
- **cloudformation/** contains the stack templates:
  - **01-base-vpc-networking.json** - VPC, public/private subnets, Internet Gateway, NAT Gateway, VPC endpoints. Exports: VPC ID, subnet IDs
  - **02-base-security-groups.json** - Security groups (ALB, ECS, RDS) and IAM roles. Exports: Security group IDs, IAM role ARNs
  - **03-data-rds.json** - RDS instance, DB subnet group, Secrets Manager credentials. Exports: Database endpoint
  - **04-compute-ecs-cluster.json** - ECS cluster, ECR repositories, IAM roles, CloudWatch log groups. Exports: Cluster name, ECR URIs
  - **05-compute-alb.json** - Application Load Balancer, target groups, SSL certificate, listener rules. Exports: ALB DNS name
  - **06-compute-fargate-scaling.json** - Auto-scaling policies for ECS services. Deploy after services are already running

## Secrets Management

**setup-secrets.sh** creates secrets in AWS Secrets Manager during initial setup. Run once per environment.

**resolve-secrets.sh** replaces secret placeholders with actual ARNs before every deployment. AWS adds random suffixes to secret ARNs:

Task definition template:

```json
{
  "secrets": [
    {
      "name": "ConnectionString",
      "valueFrom": "myproject/${ENVIRONMENT}/connection-string"
    }
  ]
}

```

ECS needs the full ARN including random suffix:

```json
{
  "secrets": [
    {
      "name": "ConnectionString",
      "valueFrom": "arn:aws:secretsmanager:eu-central-1:123456789:secret:myproject/test/connection-string-AbCdEf"
    }
  ]
}

```

resolve-secrets.sh queries AWS for actual ARNs and updates task definitions. Runs automatically in CI/CD.

## Deploying the Infrastructure

```bash
cd infrastructure/scripts
./deploy-environment.sh test myproject-infra

```

The script:

- Validates all templates
- Deploys stacks 01-05 in order, and waits for each stack to complete
- Uses parameter file from `parameters/test.json` or `parameters/prod.json`
- Outputs DNS configuration info

After infrastructure deployment:

1. Run `setup-secrets.sh` to create placeholder secrets
2. Update secrets with actual values via AWS CLI or Console
3. Deploy ECS services via GitHub Actions
4. Run `deploy-scaling.sh` to configure auto-scaling

## Deploying the Application

Application deployment is separate from infrastructure. Infrastructure changes infrequently. Code changes constantly.

GitHub Actions workflow:

1. Build Docker images
2. Push to ECR
3. Run resolve-secrets.sh on task definitions
4. Update ECS task definitions with new image tags
5. Deploy to ECS
6. Wait for deployment completion

## Cleanup

Delete stacks in reverse order (06 → 01):

```bash
aws cloudformation delete-stack --stack-name myproject-test-compute-fargate-scaling
aws cloudformation delete-stack --stack-name myproject-test-compute-alb
aws cloudformation delete-stack --stack-name myproject-test-compute-ecs-cluster
aws cloudformation delete-stack --stack-name myproject-test-data-rds
aws cloudformation delete-stack --stack-name myproject-test-base-security-groups
aws cloudformation delete-stack --stack-name myproject-test-base-vpc-networking

```

Manual cleanup needed: S3 buckets (CloudFormation won't delete non-empty buckets), ECR images, CloudWatch log groups with never-expire retention, Secrets Manager secrets (deletion protection enabled).
