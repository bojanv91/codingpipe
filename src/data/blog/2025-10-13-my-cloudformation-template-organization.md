---
title: "A Baseline CloudFormation Layout That I Keep Reusing"
pubDatetime: 2025-10-13
description: "A structured approach to organizing AWS CloudFormation templates for containerized web applications, covering VPC networking, ECS Fargate, RDS, and automated deployment processes."
slug: a-baseline-cloudformation-layout-that-i-keep-reusing
tags:
  - devops
draft: false
---

I needed a repeatable way to deploy containerized web apps on AWS. This is the baseline I settled on — VPC networking, ECS Fargate, RDS, load balancing, auto-scaling, separate test and prod environments. I modify it per project.

**Template structure:**

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

**Numbering enforces deployment order.** CloudFormation stacks have dependencies. You can't create an ECS service before the VPC exists. The numbered prefix makes the sequence explicit.

**Traffic flow:** Internet → ALB (public subnets) → ECS tasks (private subnets) → RDS (private subnets). ECS pulls images from ECR via VPC endpoint. NAT Gateway handles outbound traffic from private subnets.

**parameters/** contains environment-specific values: CIDR blocks, instance sizes, database credentials — anything that differs between test and prod.

**scripts/** automates deployment. `deploy-environment.sh` deploys stacks 01–05 in sequence. `setup-secrets.sh` creates secrets in AWS Secrets Manager during initial setup. `resolve-secrets.sh` replaces secret placeholders with actual ARNs before every deployment. `deploy-scaling.sh` deploys auto-scaling after services are running.

**cloudformation/** contains the stack templates:

- `01-base-vpc-networking.json` — VPC, public/private subnets, Internet Gateway, NAT Gateway, VPC endpoints. Exports: VPC ID, subnet IDs.
- `02-base-security-groups.json` — Security groups (ALB, ECS, RDS) and IAM roles. Exports: security group IDs, IAM role ARNs.
- `03-data-rds.json` — RDS instance, DB subnet group, Secrets Manager credentials. Exports: database endpoint.
- `04-compute-ecs-cluster.json` — ECS cluster, ECR repositories, IAM roles, CloudWatch log groups. Exports: cluster name, ECR URIs.
- `05-compute-alb.json` — ALB, target groups, SSL certificate, listener rules. Exports: ALB DNS name.
- `06-compute-fargate-scaling.json` — Auto-scaling policies for ECS services. Deploy after services are running.

**Secrets management:** AWS adds random suffixes to secret ARNs. Task definitions reference secrets by logical name; ECS needs the full ARN.

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

After `resolve-secrets.sh` runs:

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

`resolve-secrets.sh` queries AWS for actual ARNs and updates task definitions. It runs automatically in CI/CD.

**Deploying the infrastructure:**

```bash
cd infrastructure/scripts
./deploy-environment.sh test myproject-infra
```

The script validates all templates, deploys stacks 01–05 in order, waits for each to complete, and uses the parameter file from `parameters/test.json` or `parameters/prod.json`.

After infrastructure deployment:

1. Run `setup-secrets.sh` to create placeholder secrets.
2. Update secrets with actual values via AWS CLI or Console.
3. Deploy ECS services via GitHub Actions.
4. Run `deploy-scaling.sh` to configure auto-scaling.

**Deploying the application:** Application deployment is separate from infrastructure. Infrastructure changes infrequently. Code changes constantly.

GitHub Actions workflow:

1. Build Docker images.
2. Push to ECR.
3. Run `resolve-secrets.sh` on task definitions.
4. Update ECS task definitions with new image tags.
5. Deploy to ECS and wait for completion.

**Cleanup:** Delete stacks in reverse order (06 → 01):

```bash
aws cloudformation delete-stack --stack-name myproject-test-compute-fargate-scaling
aws cloudformation delete-stack --stack-name myproject-test-compute-alb
aws cloudformation delete-stack --stack-name myproject-test-compute-ecs-cluster
aws cloudformation delete-stack --stack-name myproject-test-data-rds
aws cloudformation delete-stack --stack-name myproject-test-base-security-groups
aws cloudformation delete-stack --stack-name myproject-test-base-vpc-networking
```

Manual cleanup required: S3 buckets (CloudFormation won't delete non-empty buckets), ECR images, CloudWatch log groups with never-expire retention, Secrets Manager secrets (deletion protection enabled).

---

**TLDR:** Six numbered CloudFormation stacks, deployed in order, with environment parameters split into separate files. Secrets management is its own step — `resolve-secrets.sh` bridges the gap between logical names and ARNs with random suffixes. Application deployment is a separate pipeline from infrastructure.

This structure provides a repeatable, maintainable baseline for containerized web apps on AWS. I modify it per project, but the core organization remains consistent.
