# Deployment & Network Standards

> **TODO:** Replace this placeholder with your organization's deployment and network standards.
> This file is read by the `deployment-patterns-skill` to enforce approved patterns.

---

## Compute Patterns

> **TODO:** Define approved compute options for each workload type.

| Workload | Approved Compute | Notes |
|---------|-----------------|-------|
| Long-running API / service | [e.g., EKS / ECS Fargate] | [constraints] |
| Scheduled / batch job | [e.g., ECS Scheduled Task] | [constraints] |
| Event-triggered function | [e.g., Lambda] | [max duration, memory limits] |
| Static UI / MFE | [e.g., S3 + CloudFront] | [CDN config requirements] |
| Data pipeline | [e.g., Glue / EMR] | [constraints] |

---

## Deployment Tooling

> **TODO:** Define approved tooling for each function.

| Function | Approved Tool | Version / Notes |
|---------|--------------|----------------|
| Infrastructure as Code | [e.g., Terraform, CDK] | [version + module standards] |
| CI/CD pipeline | [e.g., GitHub Actions] | [pipeline template link] |
| Container registry | [e.g., ECR] | [registry URL + naming convention] |
| Secret management | [e.g., AWS Secrets Manager] | [rotation policy] |
| Configuration | [e.g., Parameter Store, ConfigMap] | [conventions] |

---

## Environment Strategy

> **TODO:** Define the environment topology.

| Environment | Purpose | AWS Account | Promotion Gate |
|-------------|---------|-------------|----------------|
| Dev | Developer integration | [Account ID] | Automated tests pass |
| Test / QA | Functional + regression | [Account ID] | QA sign-off |
| Staging | Production parity | [Account ID] | Perf test + approval |
| Production | Live | [Account ID] | Change board + approval |

---

## Network Standards

> **TODO:** Define VPC, subnet, and connectivity conventions.

### VPC Design
- Number of VPCs: [e.g., one per account, shared services VPC pattern]
- VPC CIDR ranges: [conventions / allocation process]
- VPC peering / Transit Gateway: [how VPCs connect]

### Subnet Tiers

| Tier | Type | What belongs here |
|------|------|------------------|
| Public | Internet-facing | Load balancers, NAT Gateway |
| Private | No direct internet | Application services, EKS nodes |
| Isolated | No internet, no NAT | Databases, cache, sensitive data stores |

### Security Groups
- One security group per service (not shared across services)
- Inbound: allow only from known sources (no 0.0.0.0/0 on internal services)
- Outbound: restrict to required destinations where feasible
- Naming convention: `[app-id]-[env]-[component]-sg`

### Load Balancing
- External traffic: Application Load Balancer (ALB) with WAF
- Internal service-to-service: Internal ALB or service mesh
- TCP/TLS passthrough: Network Load Balancer (NLB)

### DNS Conventions
- Internal services: [e.g., `service-name.internal.company.com`]
- External APIs: [e.g., `api.company.com/service-name`]
- Service discovery: [e.g., AWS Cloud Map, CoreDNS]

---

## Exception Process

> **TODO:** Document the exception process for unsupported patterns.

1. Submit to: [Architecture Review Board]
2. Required: proposed pattern, reason approved options don't fit, risk assessment
3. Review SLA: [X business days]
