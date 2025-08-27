# Project 19: AI Governance Framework

## Overview

HashiCorp Sentinel policies and OPA Gatekeeper implementation for ML model governance. Infrastructure policies enforced through HCP Terraform. Runtime policies enforced on AWS EKS cluster.

## Architecture

```
GitHub → HCP Terraform → Sentinel Policies → AWS EKS → OPA Gatekeeper → Kubernetes
         (VCS)          (Pre-deployment)    (Infrastructure)  (Runtime)     (ML Workloads)
```

## Technologies Used

- HashiCorp Terraform Cloud - Infrastructure automation and state management
- HashiCorp Sentinel - Policy as Code for infrastructure
- AWS EKS - Managed Kubernetes service
- OPA Gatekeeper - Kubernetes admission controller
- GitHub - Version control and GitOps workflow

## Project Structure

```
project-19-ai-governance/
├── terraform/                    # Infrastructure as Code
│   ├── main.tf                  # EKS cluster configuration
│   ├── variables.tf             # Variable definitions
│   └── outputs.tf               # Output values
├── sentinel-policies/            # Infrastructure Governance
│   ├── sentinel.hcl            
│   ├── gpu-instance-control.sentinel      # Instance type restrictions
│   ├── ai-resource-tagging.sentinel       # Resource tagging enforcement
│   ├── ai-spending-limits.sentinel        # Cost control policies
│   └── model-deployment-rules.sentinel    # Deployment requirements
└── opa-policies/                # Runtime Governance
    ├── templates/
    │   ├── require-labels-template.yaml        # Label requirements
    │   └── ml-model-registry-template.yaml     # Registry enforcement
    ├── constraints/
    │   ├── ml-model-governance.yaml            # Label enforcement
    │   └── ml-registry-requirement.yaml        # Registry requirements
    └── deployments/
        ├── valid-deployment.yaml               # Compliant deployment example
        └── invalid-deployment.yaml             # Non-compliant example
```

## Implementation

### Infrastructure Policies (Sentinel)

Sentinel policies validate Terraform configurations before deployment:
- Instance type restrictions (preventing specific instance types)
- Resource tagging requirements (enforcing label standards)
- Spending limits (setting infrastructure cost boundaries)
- Deployment rules (requiring specific configurations)

### Runtime Policies (OPA Gatekeeper)

OPA Gatekeeper enforces policies at the Kubernetes API level:
- Label requirements for deployments
- Model registry URL validation
- Namespace restrictions
- Resource constraints

### Policy Examples

Sentinel policy for instance type control:
```hcl
allowed_instance_types = [
  "t3.micro",
  "t3.small",
  "t3.medium"
]

main = rule {
  all ec2_instances as _, instance {
    instance.change.after.instance_type in allowed_instance_types
  }
}
```

OPA constraint for label enforcement:
```yaml
violation[{"msg": msg}] {
  required := input.parameters.labels
  provided := input.review.object.metadata.labels
  missing := [label | required[_] = label; not provided[label]]
  
  msg := sprintf("Missing Labels: %v", [missing])
}
```

## Screenshots

1. [VCS provider configuration in HCP Terraform](screenshots/1.png)
2. [Policy set connected to workspace](screenshots/2.png)
3. [Terraform infrastructure deployment (77 resources)](screenshots/3.png)
4. [Sentinel policies evaluation results](screenshots/4.png)
5. [AWS EKS cluster with nodes](screenshots/5.png)
6. [OPA Gatekeeper blocking non-compliant deployment](screenshots/6.png)
7. [OPA Gatekeeper allowing compliant deployment](screenshots/7.png)
8. [Model serving endpoint](screenshots/8.png)
9. [All platform components running](screenshots/9.png)
10. [Infrastructure teardown](screenshots/10.png)

## Deployment Process

1. GitHub repository connected as VCS provider
2. Sentinel policies configured in HCP Terraform
3. Terraform creates EKS cluster and supporting infrastructure
4. OPA Gatekeeper deployed to cluster
5. Constraint templates and constraints applied
6. Test deployments validate policy enforcement

## Features

### Pre-Deployment Governance
- Instance type restrictions
- Resource tagging enforcement
- Cost control policies
- Deployment configuration requirements

### Runtime Governance
- Kubernetes admission control
- Label enforcement for deployments
- Registry URL validation
- Clear policy violation messages

### Infrastructure Components
- VPC with public and private subnets
- EKS cluster with managed node groups
- S3 bucket for model storage
- KMS encryption keys
- IAM roles and policies

## Resources Created

- 77 Terraform resources including:
  - VPC and networking components
  - EKS cluster and node groups
  - S3 bucket with encryption
  - IAM roles and policies
  - Security groups

## Policy Enforcement

The framework demonstrates policy enforcement at two levels:
1. Infrastructure provisioning through Sentinel
2. Kubernetes deployments through OPA Gatekeeper

Both layers work together to enforce governance requirements throughout the deployment lifecycle.