# IAC-Spec-Kit: Advanced Infrastructure Specifications

> **Create detailed, validated specifications for better infrastructure code generation**

## 📋 Overview

IAC-Spec-Kit is a powerful framework for creating comprehensive infrastructure specifications that Bob can use to generate more accurate, complete, and maintainable Terraform code. Think of it as a blueprint language for your cloud infrastructure.

**What You'll Learn:**
- Understanding IAC-Spec-Kit concepts
- Writing infrastructure specifications
- Validation and testing specs
- Generating Terraform from specs
- Advanced specification patterns
- Team collaboration with specs
- Spec versioning and evolution

**Prerequisites:**
- Completed all previous guides
- Experience with Terraform and Bob
- Understanding of infrastructure patterns

**Estimated Time:** 45-60 minutes

---

## 🎯 Why Use IAC-Spec-Kit?

### The Problem with Ad-Hoc Generation

**Traditional Approach:**
```bash
bob> Create a VPC with some subnets and a database
```

**Issues:**
- ❌ Vague requirements lead to assumptions
- ❌ Missing important details
- ❌ Hard to review and validate
- ❌ Difficult to version and track changes
- ❌ No validation before generation

### The IAC-Spec-Kit Solution

**Specification-Driven Approach:**
```yaml
# infrastructure-spec.yaml
version: "1.0"
metadata:
  name: "production-infrastructure"
  description: "Complete production environment"
  owner: "platform-team"
  
infrastructure:
  vpc:
    name: "production-vpc"
    region: "us-south"
    address_prefixes:
      - "10.240.0.0/16"
    subnets:
      - name: "public-subnet-1"
        zone: "us-south-1"
        cidr: "10.240.1.0/24"
        public: true
      - name: "private-subnet-1"
        zone: "us-south-1"
        cidr: "10.240.2.0/24"
        public: false
    
  database:
    type: "postgresql"
    version: "14"
    plan: "standard"
    memory: "8GB"
    disk: "100GB"
    high_availability: true
    backup_retention: 30
```

**Benefits:**
- ✅ Clear, explicit requirements
- ✅ Validated before generation
- ✅ Version controlled
- ✅ Easy to review and approve
- ✅ Consistent across teams
- ✅ Self-documenting

---

## 🏗️ IAC-Spec-Kit Structure

### Basic Specification Format

```yaml
# Complete specification structure
version: "1.0"

metadata:
  name: "infrastructure-name"
  description: "Description of the infrastructure"
  owner: "team-name"
  environment: "production"
  created: "2026-01-06"
  updated: "2026-01-06"
  tags:
    - "key:value"

requirements:
  compliance:
    - "SOC2"
    - "HIPAA"
  security_level: "high"
  availability: "99.99%"
  disaster_recovery: true

infrastructure:
  # Resource definitions
  
validation:
  # Validation rules
  
outputs:
  # Expected outputs
```

### Metadata Section

```yaml
metadata:
  name: "production-webapp"
  description: "Production infrastructure for web application"
  owner: "platform-team"
  contact: "platform@example.com"
  environment: "production"
  region: "us-south"
  
  # Version tracking
  version: "2.1.0"
  created: "2025-12-01"
  updated: "2026-01-06"
  
  # Classification
  tags:
    cost-center: "engineering"
    project: "webapp-v2"
    compliance: "SOC2"
    data-classification: "confidential"
  
  # Dependencies
  depends_on:
    - "shared-services-vpc"
    - "key-protect-instance"
```

### Requirements Section

```yaml
requirements:
  # Compliance requirements
  compliance:
    standards:
      - "SOC2"
      - "GDPR"
      - "HIPAA"
    encryption:
      at_rest: true
      in_transit: true
      key_management: "key-protect"
    audit_logging: true
  
  # Availability requirements
  availability:
    target: "99.99%"
    multi_zone: true
    disaster_recovery: true
    backup_frequency: "daily"
    backup_retention_days: 30
  
  # Security requirements
  security:
    level: "high"
    network_isolation: true
    private_endpoints: true
    security_scanning: true
    vulnerability_management: true
  
  # Performance requirements
  performance:
    response_time_ms: 200
    throughput_rps: 10000
    concurrent_users: 5000
  
  # Cost constraints
  cost:
    monthly_budget: 5000
    cost_optimization: true
    reserved_capacity: true
```

---

## 📝 Writing Infrastructure Specifications

### Example 1: VPC Specification

```yaml
version: "1.0"

metadata:
  name: "production-vpc"
  description: "Production VPC with multi-zone deployment"
  owner: "platform-team"
  environment: "production"

requirements:
  availability:
    multi_zone: true
    zones: 3
  security:
    level: "high"
    private_endpoints: true

infrastructure:
  vpc:
    name: "production-vpc"
    region: "us-south"
    resource_group: "production"
    
    # Address prefixes
    address_prefixes:
      - cidr: "10.240.0.0/16"
        zone: "us-south-1"
      - cidr: "10.241.0.0/16"
        zone: "us-south-2"
      - cidr: "10.242.0.0/16"
        zone: "us-south-3"
    
    # Public subnets (one per zone)
    public_subnets:
      - name: "public-subnet-1"
        zone: "us-south-1"
        cidr: "10.240.1.0/24"
        public_gateway: true
      - name: "public-subnet-2"
        zone: "us-south-2"
        cidr: "10.241.1.0/24"
        public_gateway: true
      - name: "public-subnet-3"
        zone: "us-south-3"
        cidr: "10.242.1.0/24"
        public_gateway: true
    
    # Private subnets (one per zone)
    private_subnets:
      - name: "private-subnet-1"
        zone: "us-south-1"
        cidr: "10.240.2.0/24"
      - name: "private-subnet-2"
        zone: "us-south-2"
        cidr: "10.241.2.0/24"
      - name: "private-subnet-3"
        zone: "us-south-3"
        cidr: "10.242.2.0/24"
    
    # Security groups
    security_groups:
      - name: "load-balancer-sg"
        rules:
          - direction: "inbound"
            protocol: "tcp"
            port_min: 443
            port_max: 443
            source: "0.0.0.0/0"
          - direction: "inbound"
            protocol: "tcp"
            port_min: 80
            port_max: 80
            source: "0.0.0.0/0"
      
      - name: "application-sg"
        rules:
          - direction: "inbound"
            protocol: "tcp"
            port_min: 8080
            port_max: 8080
            source_security_group: "load-balancer-sg"
      
      - name: "database-sg"
        rules:
          - direction: "inbound"
            protocol: "tcp"
            port_min: 5432
            port_max: 5432
            source_security_group: "application-sg"
    
    # Network ACLs
    network_acls:
      - name: "public-acl"
        rules:
          - name: "allow-inbound-http"
            action: "allow"
            direction: "inbound"
            source: "0.0.0.0/0"
            destination: "0.0.0.0/0"
            protocol: "tcp"
            port_min: 80
            port_max: 80
          - name: "allow-inbound-https"
            action: "allow"
            direction: "inbound"
            source: "0.0.0.0/0"
            destination: "0.0.0.0/0"
            protocol: "tcp"
            port_min: 443
            port_max: 443

validation:
  rules:
    - name: "subnet-count"
      condition: "len(public_subnets) == len(private_subnets)"
      message: "Must have equal number of public and private subnets"
    
    - name: "cidr-no-overlap"
      condition: "no_cidr_overlap(all_subnets)"
      message: "Subnet CIDR blocks must not overlap"
    
    - name: "multi-zone"
      condition: "len(unique_zones) >= 3"
      message: "Must span at least 3 availability zones"

outputs:
  vpc_id:
    description: "ID of the created VPC"
  public_subnet_ids:
    description: "List of public subnet IDs"
  private_subnet_ids:
    description: "List of private subnet IDs"
  security_group_ids:
    description: "Map of security group names to IDs"
```

### Example 2: Kubernetes Cluster Specification

```yaml
version: "1.0"

metadata:
  name: "production-kubernetes"
  description: "Production Kubernetes cluster"
  owner: "platform-team"
  environment: "production"

requirements:
  availability:
    target: "99.99%"
    multi_zone: true
  security:
    level: "high"
    private_endpoint: true
  performance:
    node_count_min: 3
    node_count_max: 10

infrastructure:
  kubernetes:
    name: "production-cluster"
    version: "1.28"
    region: "us-south"
    vpc_id: "${vpc.production-vpc.id}"
    resource_group: "production"
    
    # Service endpoint configuration
    service_endpoint:
      public: false
      private: true
    
    # Default worker pool
    default_worker_pool:
      name: "default"
      flavor: "bx2.4x16"
      workers_per_zone: 2
      
      zones:
        - name: "us-south-1"
          subnet_id: "${vpc.production-vpc.private_subnet_1}"
        - name: "us-south-2"
          subnet_id: "${vpc.production-vpc.private_subnet_2}"
        - name: "us-south-3"
          subnet_id: "${vpc.production-vpc.private_subnet_3}"
    
    # Additional worker pools
    worker_pools:
      - name: "compute-intensive"
        flavor: "bx2.8x32"
        workers_per_zone: 1
        labels:
          workload: "compute"
        taints:
          - key: "compute"
            value: "true"
            effect: "NoSchedule"
      
      - name: "memory-intensive"
        flavor: "mx2.8x64"
        workers_per_zone: 1
        labels:
          workload: "memory"
        taints:
          - key: "memory"
            value: "true"
            effect: "NoSchedule"
    
    # Add-ons
    addons:
      - name: "vpc-block-csi-driver"
        version: "5.0"
      - name: "cluster-autoscaler"
        version: "1.0.7"
        config:
          min_size: 3
          max_size: 10
    
    # Logging and monitoring
    logging:
      enabled: true
      instance_id: "${logs.production-logs.id}"
    
    monitoring:
      enabled: true
      instance_id: "${monitoring.production-monitoring.id}"

validation:
  rules:
    - name: "min-workers"
      condition: "total_workers >= 3"
      message: "Must have at least 3 worker nodes"
    
    - name: "multi-zone"
      condition: "len(zones) >= 3"
      message: "Must span at least 3 zones"
    
    - name: "private-endpoint"
      condition: "service_endpoint.private == true"
      message: "Private endpoint must be enabled"
```

### Example 3: Database Specification

```yaml
version: "1.0"

metadata:
  name: "production-database"
  description: "Production PostgreSQL database"
  owner: "platform-team"
  environment: "production"

requirements:
  availability:
    high_availability: true
    backup_retention_days: 30
  security:
    encryption: true
    private_endpoint: true
  performance:
    memory_gb: 16
    disk_gb: 500
    iops: 10000

infrastructure:
  database:
    name: "production-postgresql"
    service: "databases-for-postgresql"
    version: "14"
    plan: "standard"
    region: "us-south"
    resource_group: "production"
    
    # Resource allocation
    resources:
      members: 3  # 1 primary + 2 replicas
      memory_mb_per_member: 5461  # ~16GB total
      disk_mb_per_member: 170667  # ~500GB total
      cpu_count_per_member: 3
    
    # Network configuration
    service_endpoints: "private"
    
    # Encryption
    encryption:
      key_protect_instance: "${key_protect.production.id}"
      key_protect_key: "${key_protect.production.database_key}"
    
    # Backup configuration
    backup:
      enabled: true
      retention_days: 30
      point_in_time_recovery: true
    
    # Connection configuration
    configuration:
      max_connections: 200
      shared_buffers: "4GB"
      effective_cache_size: "12GB"
      maintenance_work_mem: "1GB"
      checkpoint_completion_target: 0.9
      wal_buffers: "16MB"
      default_statistics_target: 100
      random_page_cost: 1.1
      effective_io_concurrency: 200
    
    # Allowlist (IP whitelist)
    allowlist:
      - address: "${vpc.production-vpc.private_subnet_1.cidr}"
        description: "Private subnet 1"
      - address: "${vpc.production-vpc.private_subnet_2.cidr}"
        description: "Private subnet 2"
      - address: "${vpc.production-vpc.private_subnet_3.cidr}"
        description: "Private subnet 3"
    
    # Monitoring
    monitoring:
      enabled: true
      alerts:
        - metric: "cpu_usage"
          threshold: 80
          duration: 300
        - metric: "memory_usage"
          threshold: 85
          duration: 300
        - metric: "disk_usage"
          threshold: 80
          duration: 300

validation:
  rules:
    - name: "high-availability"
      condition: "resources.members >= 3"
      message: "Must have at least 3 members for HA"
    
    - name: "private-endpoint"
      condition: "service_endpoints == 'private'"
      message: "Must use private endpoint"
    
    - name: "encryption"
      condition: "encryption.key_protect_key != null"
      message: "Must use Key Protect for encryption"
```

---

## 🔍 Using Specs with Bob

### Generate from Specification

```bash
bob

# Load and generate from spec
bob> load specification from infrastructure-spec.yaml

bob> validate the specification

bob> generate terraform code from specification

bob> review generated code for security issues

bob> apply the infrastructure
```

### Interactive Spec Creation

```bash
bob> create infrastructure specification for a 3-tier web application

# Bob will ask questions and build the spec interactively
```

**Bob's Interactive Process:**
```
I'll help you create a specification for a 3-tier web application.

1. Environment details:
   - Environment name? (e.g., production, staging): production
   - Region? (e.g., us-south, eu-de): us-south
   - High availability required? (yes/no): yes

2. Network configuration:
   - Number of availability zones? (1-3): 3
   - VPC CIDR block? (e.g., 10.240.0.0/16): 10.240.0.0/16
   - Public subnets needed? (yes/no): yes

3. Compute tier:
   - Instance type? (e.g., bx2.4x16): bx2.4x16
   - Number of instances per zone? (1-10): 2
   - Auto-scaling? (yes/no): yes

4. Database tier:
   - Database type? (postgresql/mysql/mongodb): postgresql
   - Database size? (small/medium/large): large
   - High availability? (yes/no): yes

5. Load balancer:
   - Type? (application/network): application
   - SSL certificate? (yes/no): yes

Generating specification...
```

### Spec Validation

```bash
bob> validate infrastructure-spec.yaml

# Bob checks:
# - Syntax correctness
# - Required fields present
# - Value constraints
# - Resource dependencies
# - Security requirements
# - Cost estimates
```

---

## 🎓 Advanced Specification Patterns

### 1. Modular Specifications

```yaml
# base-spec.yaml
version: "1.0"
metadata:
  name: "base-infrastructure"
  type: "base"

infrastructure:
  vpc:
    name: "${var.environment}-vpc"
    region: "${var.region}"
```

```yaml
# production-spec.yaml
version: "1.0"
extends: "base-spec.yaml"

metadata:
  name: "production-infrastructure"
  environment: "production"

variables:
  environment: "production"
  region: "us-south"

infrastructure:
  vpc:
    # Inherits from base, adds specifics
    address_prefixes:
      - "10.240.0.0/16"
```

### 2. Conditional Resources

```yaml
infrastructure:
  database:
    name: "app-database"
    
    # Conditional configuration
    when:
      environment: "production"
    then:
      resources:
        members: 3
        memory_mb_per_member: 8192
    else:
      resources:
        members: 1
        memory_mb_per_member: 2048
```

### 3. Resource Templates

```yaml
templates:
  standard_subnet:
    type: "subnet"
    properties:
      total_ipv4_address_count: 256
      public_gateway: false
      network_acl: "default"

infrastructure:
  vpc:
    subnets:
      - template: "standard_subnet"
        name: "subnet-1"
        zone: "us-south-1"
        cidr: "10.240.1.0/24"
      
      - template: "standard_subnet"
        name: "subnet-2"
        zone: "us-south-2"
        cidr: "10.240.2.0/24"
```

### 4. Dynamic Resource Generation

```yaml
infrastructure:
  vpc:
    subnets:
      # Generate 3 subnets dynamically
      count: 3
      template:
        name: "subnet-${index}"
        zone: "us-south-${index + 1}"
        cidr: "10.240.${index + 1}.0/24"
```

---

## 🧪 Testing Specifications

### Validation Rules

```yaml
validation:
  # Built-in validators
  rules:
    - name: "vpc-cidr-valid"
      type: "cidr"
      field: "infrastructure.vpc.address_prefixes[*].cidr"
      message: "Invalid CIDR block"
    
    - name: "subnet-in-vpc"
      type: "cidr_contains"
      parent: "infrastructure.vpc.address_prefixes[*].cidr"
      child: "infrastructure.vpc.subnets[*].cidr"
      message: "Subnet CIDR must be within VPC CIDR"
    
    - name: "unique-names"
      type: "unique"
      field: "infrastructure.vpc.subnets[*].name"
      message: "Subnet names must be unique"
    
    # Custom validators
    - name: "cost-limit"
      type: "custom"
      script: |
        estimated_cost = calculate_cost(infrastructure)
        return estimated_cost <= requirements.cost.monthly_budget
      message: "Estimated cost exceeds budget"
```

### Testing with Bob

```bash
bob> test specification infrastructure-spec.yaml

# Bob performs:
# 1. Syntax validation
# 2. Schema validation
# 3. Constraint checking
# 4. Dependency validation
# 5. Security scanning
# 6. Cost estimation
# 7. Dry-run generation
```

---

## 👥 Team Collaboration

### Spec Review Process

```yaml
# Add review metadata
metadata:
  review:
    status: "pending"
    reviewers:
      - "alice@example.com"
      - "bob@example.com"
    created_by: "charlie@example.com"
    created_at: "2026-01-06T10:00:00Z"
    
  approval:
    required_approvals: 2
    approvals:
      - reviewer: "alice@example.com"
        approved_at: "2026-01-06T11:00:00Z"
        comments: "LGTM, security requirements met"
```

### Using Bob for Reviews

```bash
bob> review specification infrastructure-spec.yaml

# Bob provides:
# - Security analysis
# - Cost estimation
# - Best practice compliance
# - Potential issues
# - Optimization suggestions
```

### Version Control

```bash
# Track spec changes
git add infrastructure-spec.yaml
git commit -m "feat: add production database specification"

# Use Bob to compare versions
bob> compare specification v1.0 with v2.0

bob> show what changed between commits abc123 and def456
```

---

## 📊 Spec to Terraform Workflow

### Complete Workflow

```bash
# 1. Create specification
bob> create specification for production environment

# 2. Validate specification
bob> validate infrastructure-spec.yaml

# 3. Review and approve
bob> review specification for security and cost

# 4. Generate Terraform
bob> generate terraform from infrastructure-spec.yaml

# 5. Review generated code
bob> review generated terraform code

# 6. Plan changes
bob> terraform plan

# 7. Apply infrastructure
bob> terraform apply

# 8. Verify deployment
bob> verify infrastructure matches specification
```

### Continuous Validation

```yaml
# .github/workflows/spec-validation.yml
name: Validate Infrastructure Spec

on:
  pull_request:
    paths:
      - '**/*-spec.yaml'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Bob-Shell
        run: |
          curl -fsSL https://bob-shell.dev/install.sh | sh
      
      - name: Validate Specifications
        run: |
          bob validate-all-specs
      
      - name: Estimate Costs
        run: |
          bob estimate-costs --spec infrastructure-spec.yaml
      
      - name: Security Scan
        run: |
          bob security-scan --spec infrastructure-spec.yaml
```

---

## 🎯 Best Practices for Specifications

### 1. Be Explicit

```yaml
# ❌ Vague
database:
  name: "db"
  size: "medium"

# ✅ Explicit
database:
  name: "production-postgresql"
  service: "databases-for-postgresql"
  version: "14"
  resources:
    members: 3
    memory_mb_per_member: 8192
    disk_mb_per_member: 102400
```

### 2. Document Decisions

```yaml
infrastructure:
  database:
    # Decision: Using 3 members for high availability
    # Rationale: Provides automatic failover and read scaling
    # Alternative considered: Single instance with backups
    # Trade-off: Higher cost but better availability
    resources:
      members: 3
```

### 3. Use Variables

```yaml
variables:
  environment: "production"
  region: "us-south"
  app_name: "webapp"

infrastructure:
  vpc:
    name: "${var.environment}-${var.app_name}-vpc"
    region: "${var.region}"
```

### 4. Include Validation

```yaml
validation:
  rules:
    - name: "production-requirements"
      condition: |
        if environment == "production":
          return (
            database.resources.members >= 3 and
            kubernetes.workers_per_zone >= 2 and
            vpc.zones >= 3
          )
        return true
      message: "Production must meet HA requirements"
```

---

## 🎓 Summary

IAC-Spec-Kit enables you to:

- ✅ Create detailed, validated infrastructure specifications
- ✅ Generate consistent, high-quality Terraform code
- ✅ Collaborate effectively with team reviews
- ✅ Version control infrastructure designs
- ✅ Validate before deployment
- ✅ Document architectural decisions
- ✅ Ensure compliance and security

---

## 🎯 Next Steps

You've completed the guide! Now:

1. **[Examples](../examples/)** - See complete specification examples
2. **[Contributing](../CONTRIBUTING.md)** - Share your specifications
3. **[Resources](./08-resources.md)** - Additional learning materials

---

## 📚 Additional Resources

- [IAC-Spec-Kit Documentation](https://iac-spec-kit.dev)
- [YAML Specification](https://yaml.org/spec/)
- [Infrastructure as Code Patterns](https://www.terraform.io/docs/cloud/guides/recommended-practices)

---

**Congratulations!** You've completed the IBM Cloud with Bob guide. Start building amazing infrastructure! 🚀
