# Best Practices for IBM Cloud with Bob

> **Build secure, scalable, and maintainable infrastructure with proven patterns**

## 📋 Overview

This guide compiles best practices, patterns, and recommendations for managing IBM Cloud infrastructure using Bob IDE and Bob-Shell. These practices are derived from real-world experience and IBM Cloud's architectural guidelines.

**What You'll Learn:**
- Security and compliance best practices
- Resource organization and naming conventions
- State management strategies
- Cost optimization techniques
- Monitoring and logging patterns
- Team collaboration workflows
- Performance optimization
- Disaster recovery planning

**Prerequisites:**
- Completed all previous guides
- Active IBM Cloud infrastructure
- Team collaboration experience (recommended)

**Estimated Time:** 45-60 minutes

---

## 🛡️ Security Best Practices

### 1. Credential Management

#### ❌ Never Do This

```hcl
# DON'T hardcode credentials
provider "ibm" {
  ibmcloud_api_key = "abc123-hardcoded-key"
  region           = "us-south"
}

# DON'T commit sensitive files
# terraform.tfvars with API keys
# .env files with secrets
```

#### ✅ Do This Instead

```bash
# Use environment variables
export IBMCLOUD_API_KEY="your-api-key"
export IC_REGION="us-south"

# Or use Bob's secure credential management
bob config set-credential ibm-cloud-api-key --secure

# Use secrets management services
bob> configure credentials using IBM Secrets Manager
```

```hcl
# Reference from environment
provider "ibm" {
  # Automatically uses IBMCLOUD_API_KEY
  region = var.region
}

# Or use data source for Secrets Manager
data "ibm_secrets_manager_secret" "api_key" {
  instance_id = var.secrets_manager_id
  secret_id   = var.api_key_secret_id
}
```

### 2. Network Security

#### Implement Defense in Depth

```hcl
# 1. Use private endpoints
resource "ibm_database" "postgresql" {
  service_endpoints = "private"  # No public access
}

# 2. Implement security groups with least privilege
resource "ibm_is_security_group_rule" "allow_app_traffic" {
  group     = ibm_is_security_group.app_sg.id
  direction = "inbound"
  remote    = ibm_is_security_group.lb_sg.id  # Only from LB
  
  tcp {
    port_min = 8080
    port_max = 8080
  }
}

# 3. Use Network ACLs for subnet-level protection
resource "ibm_is_network_acl" "private_acl" {
  name = "private-subnet-acl"
  vpc  = ibm_is_vpc.vpc.id
  
  rules {
    name        = "allow-internal"
    action      = "allow"
    source      = "10.0.0.0/8"
    destination = "10.0.0.0/8"
    direction   = "inbound"
  }
  
  rules {
    name        = "deny-all"
    action      = "deny"
    source      = "0.0.0.0/0"
    destination = "0.0.0.0/0"
    direction   = "inbound"
  }
}

# 4. Use VPN or Direct Link for hybrid connectivity
resource "ibm_is_vpn_gateway" "vpn" {
  name   = "corporate-vpn"
  subnet = ibm_is_subnet.private_subnet.id
  mode   = "route"
}
```

### 3. Encryption

#### Encrypt Everything

```hcl
# 1. Use Key Protect for encryption keys
resource "ibm_kms_key" "root_key" {
  instance_id  = ibm_resource_instance.key_protect.guid
  key_name     = "root-key"
  standard_key = false  # Root key for envelope encryption
}

# 2. Encrypt databases
resource "ibm_database" "postgresql" {
  key_protect_key = ibm_kms_key.root_key.id
  # Encryption at rest enabled automatically
}

# 3. Encrypt object storage
resource "ibm_cos_bucket" "encrypted_bucket" {
  bucket_name          = "encrypted-data"
  resource_instance_id = ibm_resource_instance.cos.id
  region_location      = var.region
  storage_class        = "standard"
  
  key_protect = ibm_kms_key.root_key.crn
}

# 4. Encrypt block storage
resource "ibm_is_instance" "app_server" {
  boot_volume {
    name       = "encrypted-boot"
    encryption = ibm_kms_key.root_key.crn
  }
}
```

### 4. IAM and Access Control

```hcl
# Use service IDs for applications
resource "ibm_iam_service_id" "app_service_id" {
  name        = "app-service-id"
  description = "Service ID for application access"
}

# Grant minimal required permissions
resource "ibm_iam_service_policy" "app_policy" {
  iam_service_id = ibm_iam_service_id.app_service_id.id
  roles          = ["Reader"]  # Minimal access
  
  resources {
    service              = "cloud-object-storage"
    resource_instance_id = ibm_resource_instance.cos.guid
  }
}

# Use access groups for team management
resource "ibm_iam_access_group" "developers" {
  name        = "developers"
  description = "Developer team access"
}

resource "ibm_iam_access_group_policy" "dev_policy" {
  access_group_id = ibm_iam_access_group.developers.id
  roles           = ["Editor"]
  
  resources {
    resource_group_id = data.ibm_resource_group.dev.id
  }
}
```

---

## 📁 Resource Organization

### 1. Naming Conventions

#### Standard Naming Pattern

```
{environment}-{application}-{resource-type}-{region}-{instance}

Examples:
- prod-webapp-vpc-us-south-01
- staging-api-cluster-eu-de-01
- dev-database-postgresql-us-south-01
```

#### Implementation

```hcl
# Use locals for consistent naming
locals {
  environment = "production"
  application = "webapp"
  region      = "us-south"
  
  name_prefix = "${local.environment}-${local.application}"
  
  common_tags = [
    "env:${local.environment}",
    "app:${local.application}",
    "managed-by:terraform",
    "team:platform"
  ]
}

# Apply to resources
resource "ibm_is_vpc" "vpc" {
  name = "${local.name_prefix}-vpc-${local.region}"
  tags = local.common_tags
}

resource "ibm_is_subnet" "subnet" {
  count = 3
  name  = "${local.name_prefix}-subnet-${local.region}-${count.index + 1}"
  vpc   = ibm_is_vpc.vpc.id
  zone  = data.ibm_is_zones.regional.zones[count.index]
  tags  = local.common_tags
}
```

### 2. Resource Tagging Strategy

```hcl
# Comprehensive tagging
locals {
  tags = {
    # Environment
    "env:${var.environment}"
    
    # Application
    "app:${var.application_name}"
    
    # Ownership
    "team:${var.team_name}"
    "owner:${var.owner_email}"
    
    # Management
    "managed-by:terraform"
    "created-by:bob"
    
    # Cost tracking
    "cost-center:${var.cost_center}"
    "project:${var.project_name}"
    
    # Compliance
    "compliance:${var.compliance_level}"
    "data-classification:${var.data_classification}"
    
    # Lifecycle
    "lifecycle:${var.lifecycle_stage}"
    "backup:${var.backup_required}"
  }
}
```

### 3. Resource Groups

```hcl
# Organize by environment and application
resource "ibm_resource_group" "production" {
  name = "production-webapp"
}

resource "ibm_resource_group" "staging" {
  name = "staging-webapp"
}

resource "ibm_resource_group" "development" {
  name = "development-webapp"
}

# Use in resources
resource "ibm_is_vpc" "vpc" {
  name           = "production-vpc"
  resource_group = ibm_resource_group.production.id
}
```

---

## 🗂️ Project Structure

### Recommended Structure

```
ibm-cloud-infrastructure/
├── environments/
│   ├── production/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── backend.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   │   └── ...
│   └── development/
│       └── ...
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── README.md
│   ├── kubernetes/
│   │   └── ...
│   ├── database/
│   │   └── ...
│   └── monitoring/
│       └── ...
├── shared/
│   ├── key-protect.tf
│   ├── iam.tf
│   └── resource-groups.tf
├── scripts/
│   ├── deploy.sh
│   ├── destroy.sh
│   └── validate.sh
├── docs/
│   ├── architecture.md
│   ├── runbooks/
│   └── decisions/
├── .gitignore
├── .bobignore
├── AGENTS.md
└── README.md
```

### Module Best Practices

```hcl
# modules/vpc/main.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    ibm = {
      source  = "IBM-Cloud/ibm"
      version = "~> 1.50.0"
    }
  }
}

# Input validation
variable "vpc_name" {
  description = "Name of the VPC"
  type        = string
  
  validation {
    condition     = length(var.vpc_name) > 0 && length(var.vpc_name) <= 63
    error_message = "VPC name must be between 1 and 63 characters"
  }
}

variable "subnet_count" {
  description = "Number of subnets to create"
  type        = number
  default     = 3
  
  validation {
    condition     = var.subnet_count >= 1 && var.subnet_count <= 15
    error_message = "Subnet count must be between 1 and 15"
  }
}

# Clear outputs
output "vpc_id" {
  description = "ID of the created VPC"
  value       = ibm_is_vpc.vpc.id
}

output "subnet_ids" {
  description = "List of subnet IDs"
  value       = ibm_is_subnet.subnet[*].id
}
```

---

## 💾 State Management

### 1. Remote State with IBM Cloud Object Storage

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket                      = "terraform-state-production"
    key                         = "vpc/terraform.tfstate"
    region                      = "us-south"
    endpoint                    = "s3.us-south.cloud-object-storage.appdomain.cloud"
    skip_credentials_validation = true
    skip_region_validation      = true
    skip_metadata_api_check     = true
  }
}
```

### 2. State Locking

```hcl
# Use DynamoDB-compatible locking
terraform {
  backend "s3" {
    # ... other config ...
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

### 3. State File Organization

```
terraform-state-bucket/
├── production/
│   ├── vpc/terraform.tfstate
│   ├── kubernetes/terraform.tfstate
│   └── database/terraform.tfstate
├── staging/
│   └── ...
└── development/
    └── ...
```

### 4. Using Bob for State Management

```bash
bob> configure remote state using IBM Cloud Object Storage

bob> migrate local state to remote backend

bob> verify state integrity

bob> backup current state before major changes
```

---

## 💰 Cost Optimization

### 1. Right-Sizing Resources

```bash
# Use Bob to analyze and optimize
bob> analyze resource utilization and suggest right-sizing

bob> estimate cost impact of scaling down underutilized resources

bob> compare costs between different instance profiles
```

### 2. Use Reserved Capacity

```hcl
# For predictable workloads
resource "ibm_container_vpc_cluster" "cluster" {
  name         = "production-cluster"
  flavor       = "bx2.4x16"
  worker_count = 3
  
  # Use reserved capacity for cost savings
  # Configure through IBM Cloud console or API
}
```

### 3. Implement Auto-Scaling

```hcl
# Kubernetes auto-scaling
resource "ibm_container_vpc_worker_pool" "pool" {
  cluster          = ibm_container_vpc_cluster.cluster.id
  worker_pool_name = "autoscale-pool"
  flavor           = "bx2.4x16"
  
  # Auto-scale based on demand
  zones {
    subnet_id = ibm_is_subnet.subnet.id
    name      = "us-south-1"
  }
  
  # Configure auto-scaler
  worker_count = 2  # Minimum
  # Maximum configured via cluster auto-scaler
}
```

### 4. Use Spot Instances

```hcl
# For non-critical workloads
resource "ibm_is_instance" "batch_processor" {
  name    = "batch-processor"
  profile = "cx2-2x4"
  
  # Use spot instances for cost savings
  # Note: Spot instances may be reclaimed
  lifecycle {
    create_before_destroy = true
  }
}
```

### 5. Implement Resource Lifecycle

```hcl
# Object storage lifecycle
resource "ibm_cos_bucket" "logs" {
  bucket_name = "application-logs"
  
  # Transition to cheaper storage after 30 days
  archive_rule {
    rule_id = "archive-old-logs"
    enable  = true
    days    = 30
    type    = "Glacier"
  }
  
  # Delete after 90 days
  expire_rule {
    rule_id = "delete-old-logs"
    enable  = true
    days    = 90
  }
}
```

---

## 📊 Monitoring and Logging

### 1. Centralized Logging

```hcl
# IBM Cloud Logs integration
resource "ibm_resource_instance" "logs" {
  name              = "application-logs"
  service           = "logdna"
  plan              = "7-day"
  location          = var.region
  resource_group_id = data.ibm_resource_group.group.id
}

# Configure log sources
resource "ibm_resource_key" "logs_key" {
  name                 = "logs-key"
  resource_instance_id = ibm_resource_instance.logs.id
  role                 = "Manager"
}

# Use in applications
locals {
  logs_ingestion_key = ibm_resource_key.logs_key.credentials["ingestion_key"]
}
```

### 2. Monitoring Setup

```hcl
# IBM Cloud Monitoring
resource "ibm_resource_instance" "monitoring" {
  name              = "application-monitoring"
  service           = "sysdig-monitor"
  plan              = "graduated-tier"
  location          = var.region
  resource_group_id = data.ibm_resource_group.group.id
}

# Configure alerts
resource "ibm_resource_key" "monitoring_key" {
  name                 = "monitoring-key"
  resource_instance_id = ibm_resource_instance.monitoring.id
  role                 = "Manager"
}
```

### 3. Using Bob for Monitoring

```bash
bob> set up monitoring for all production resources

bob> create alerts for high CPU usage (>80%)

bob> configure log aggregation from all services

bob> create dashboard for application metrics
```

---

## 🔄 CI/CD Integration

### 1. GitHub Actions Workflow

```yaml
# .github/workflows/terraform.yml
name: Terraform CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  TF_VERSION: 1.6.0

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: ${{ env.TF_VERSION }}
      
      - name: Terraform Format
        run: terraform fmt -check -recursive
      
      - name: Terraform Init
        run: terraform init
        env:
          IBMCLOUD_API_KEY: ${{ secrets.IBMCLOUD_API_KEY }}
      
      - name: Terraform Validate
        run: terraform validate
      
      - name: Terraform Plan
        run: terraform plan
        env:
          IBMCLOUD_API_KEY: ${{ secrets.IBMCLOUD_API_KEY }}

  deploy:
    needs: validate
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Bob-Shell
        run: |
          curl -fsSL https://bob-shell.dev/install.sh | sh
          echo "$HOME/.bob-shell/bin" >> $GITHUB_PATH
      
      - name: Deploy with Bob
        run: |
          bob deploy --environment production --auto-approve
        env:
          IBMCLOUD_API_KEY: ${{ secrets.IBMCLOUD_API_KEY }}
```

### 2. GitLab CI/CD

```yaml
# .gitlab-ci.yml
stages:
  - validate
  - plan
  - deploy

variables:
  TF_VERSION: "1.6.0"

validate:
  stage: validate
  image: hashicorp/terraform:$TF_VERSION
  script:
    - terraform fmt -check -recursive
    - terraform init
    - terraform validate

plan:
  stage: plan
  image: hashicorp/terraform:$TF_VERSION
  script:
    - terraform init
    - terraform plan -out=tfplan
  artifacts:
    paths:
      - tfplan

deploy:
  stage: deploy
  image: hashicorp/terraform:$TF_VERSION
  script:
    - terraform init
    - terraform apply tfplan
  only:
    - main
  when: manual
```

---

## 👥 Team Collaboration

### 1. Code Review Process

```bash
# Use Bob for automated reviews
bob> review this pull request for security issues

bob> check if this change follows our naming conventions

bob> estimate cost impact of these changes
```

### 2. Documentation Standards

```markdown
# Module: VPC

## Purpose
Creates a VPC with configurable subnets across multiple zones.

## Usage
```hcl
module "vpc" {
  source = "./modules/vpc"
  
  vpc_name     = "production-vpc"
  subnet_count = 3
  region       = "us-south"
}
```

## Inputs
| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| vpc_name | Name of the VPC | string | n/a | yes |
| subnet_count | Number of subnets | number | 3 | no |

## Outputs
| Name | Description |
|------|-------------|
| vpc_id | ID of the created VPC |
| subnet_ids | List of subnet IDs |

## Examples
See [examples/vpc](../../examples/vpc) for complete examples.
```

### 3. Change Management

```bash
# Use Bob for change planning
bob> create a change plan for adding 2 new subnets

bob> estimate downtime for this database upgrade

bob> generate rollback plan for this deployment
```

---

## 🚨 Disaster Recovery

### 1. Backup Strategy

```hcl
# Automated backups
resource "ibm_database" "postgresql" {
  backup_id = ibm_database_backup.postgresql_backup.backup_id
  
  # Point-in-time recovery
  point_in_time_recovery_deployment_id = ibm_database.postgresql.id
  point_in_time_recovery_time          = "2026-01-06T12:00:00Z"
}

# Object storage versioning
resource "ibm_cos_bucket" "critical_data" {
  bucket_name = "critical-data"
  
  versioning {
    enabled = true
  }
  
  # Cross-region replication
  replication_rule {
    rule_id = "replicate-to-eu"
    enable  = true
    
    destination {
      bucket_name = "critical-data-eu-backup"
      region      = "eu-de"
    }
  }
}
```

### 2. Multi-Region Deployment

```hcl
# Primary region
module "vpc_us_south" {
  source = "./modules/vpc"
  
  providers = {
    ibm = ibm.us_south
  }
  
  vpc_name = "production-vpc-us-south"
  region   = "us-south"
}

# DR region
module "vpc_eu_de" {
  source = "./modules/vpc"
  
  providers = {
    ibm = ibm.eu_de
  }
  
  vpc_name = "production-vpc-eu-de"
  region   = "eu-de"
}

# Global load balancer for failover
resource "ibm_cis_global_load_balancer" "app" {
  cis_id           = ibm_cis.instance.id
  domain_id        = ibm_cis_domain.domain.id
  name             = "app.example.com"
  fallback_pool_id = ibm_cis_origin_pool.eu_de.id
  default_pool_ids = [ibm_cis_origin_pool.us_south.id]
}
```

### 3. Using Bob for DR

```bash
bob> create disaster recovery plan for production environment

bob> test failover to DR region

bob> generate runbook for disaster recovery procedures

bob> verify backup integrity for all databases
```

---

## 📈 Performance Optimization

### 1. Network Performance

```hcl
# Use transit gateway for multi-VPC connectivity
resource "ibm_tg_gateway" "transit_gateway" {
  name     = "production-transit-gateway"
  location = var.region
  global   = true
}

# Connect VPCs
resource "ibm_tg_connection" "vpc_connection" {
  gateway      = ibm_tg_gateway.transit_gateway.id
  network_type = "vpc"
  name         = "vpc-connection"
  network_id   = ibm_is_vpc.vpc.crn
}
```

### 2. Database Performance

```hcl
# Optimize database configuration
resource "ibm_database" "postgresql" {
  # Adequate resources
  members_memory_allocation_mb = 8192
  members_disk_allocation_mb   = 102400
  members_cpu_allocation_count = 6
  
  # Read replicas for read-heavy workloads
  members_count = 3  # 1 primary + 2 replicas
  
  # Connection pooling
  configuration = jsonencode({
    max_connections = 200
    shared_buffers  = "2GB"
  })
}
```

### 3. Caching Strategy

```hcl
# Redis for caching
resource "ibm_database" "redis" {
  name     = "application-cache"
  service  = "databases-for-redis"
  plan     = "standard"
  location = var.region
  
  members_memory_allocation_mb = 4096
  
  # Eviction policy
  configuration = jsonencode({
    maxmemory_policy = "allkeys-lru"
  })
}
```

---

## 🎓 Summary Checklist

### Security ✅
- [ ] No hardcoded credentials
- [ ] Encryption at rest and in transit
- [ ] Private endpoints where possible
- [ ] Least privilege access
- [ ] Network segmentation
- [ ] Regular security audits

### Organization ✅
- [ ] Consistent naming conventions
- [ ] Comprehensive tagging
- [ ] Proper resource groups
- [ ] Clear project structure
- [ ] Module-based architecture

### Operations ✅
- [ ] Remote state management
- [ ] State locking enabled
- [ ] Automated backups
- [ ] Monitoring and alerting
- [ ] Centralized logging
- [ ] CI/CD integration

### Cost ✅
- [ ] Right-sized resources
- [ ] Auto-scaling configured
- [ ] Resource lifecycle policies
- [ ] Regular cost reviews
- [ ] Reserved capacity where appropriate

### Reliability ✅
- [ ] Multi-zone deployment
- [ ] Disaster recovery plan
- [ ] Regular backup testing
- [ ] Documented runbooks
- [ ] Automated failover

---

## 🎯 Next Steps

You've learned the best practices! Now:

1. **[IAC-Spec-Kit Guide](./06-iac-spec-kit.md)** - Create detailed specifications
2. **[Examples](../examples/)** - See best practices in action
3. **[Contributing](../CONTRIBUTING.md)** - Share your own best practices

---

## 📚 Additional Resources

- [IBM Cloud Architecture Center](https://www.ibm.com/cloud/architecture)
- [Terraform Best Practices](https://www.terraform-best-practices.com/)
- [IBM Cloud Security Best Practices](https://cloud.ibm.com/docs/security)
- [Well-Architected Framework](https://www.ibm.com/cloud/architecture/architectures/well-architected)

---

**Ready for advanced specifications?** Continue to [IAC-Spec-Kit Guide](./06-iac-spec-kit.md) →
