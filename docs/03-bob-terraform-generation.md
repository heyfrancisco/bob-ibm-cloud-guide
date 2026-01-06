# AI-Assisted Terraform Generation with Bob

> **Let AI write your infrastructure code - faster, smarter, and more secure**

## 📋 Overview

This guide demonstrates how to leverage Bob's AI capabilities to generate, refine, and optimize Terraform code for IBM Cloud. Say goodbye to manual code writing and hello to natural language infrastructure provisioning.

**What You'll Learn:**
- Using natural language to generate Terraform code
- Iterative refinement with Bob
- Best practices for AI-assisted development
- Advanced generation techniques
- Code review and validation with Bob
- Real-world examples and patterns

**Prerequisites:**
- Completed [Getting Started](./01-getting-started.md) and [Terraform Basics](./02-terraform-basics.md)
- Bob-Shell installed and configured
- IBM Cloud credentials set up

**Estimated Time:** 30-45 minutes

---

## 🤖 How Bob Generates Terraform Code

### The AI Generation Process

```
┌─────────────────────────────────────────────────────────┐
│  1. You describe infrastructure in natural language     │
│     "Create a VPC with 3 subnets in different zones"    │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│  2. Bob analyzes your request                           │
│     - Understands IBM Cloud resources                   │
│     - Considers best practices                          │
│     - Checks for security implications                  │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│  3. Bob generates complete Terraform code               │
│     - Proper resource definitions                       │
│     - Variables and outputs                             │
│     - Dependencies and relationships                    │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│  4. Bob explains the code                               │
│     - What each resource does                           │
│     - Why certain choices were made                     │
│     - Security considerations                           │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│  5. You review and refine                               │
│     - Ask for modifications                             │
│     - Request additional features                       │
│     - Optimize for your needs                           │
└─────────────────────────────────────────────────────────┘
```

### What Makes Bob Special?

- **Context-Aware**: Understands IBM Cloud specifics and limitations
- **Best Practices**: Applies security and architectural patterns automatically
- **Iterative**: Easy to refine and modify generated code
- **Educational**: Explains what it's doing and why
- **Complete**: Generates all necessary files (main.tf, variables.tf, outputs.tf)

---

## 🚀 Basic Generation Examples

### Example 1: Simple VPC

**Your Request:**
```bash
bob

bob> Create a VPC named "production-vpc" in us-south
```

**Bob's Response:**
```
I'll create a VPC in IBM Cloud for you. Here's what I'll generate:

1. VPC resource with the name "production-vpc"
2. Resource group reference
3. Proper tagging for management
4. Output values for the VPC ID

Generated Terraform code:
───────────────────────────────────────────────────────
# main.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    ibm = {
      source  = "IBM-Cloud/ibm"
      version = "~> 1.50.0"
    }
  }
}

provider "ibm" {
  region = var.region
}

# Get resource group
data "ibm_resource_group" "group" {
  name = var.resource_group_name
}

# Create VPC
resource "ibm_is_vpc" "production_vpc" {
  name           = "production-vpc"
  resource_group = data.ibm_resource_group.group.id
  
  tags = [
    "env:production",
    "managed-by:terraform",
    "created-by:bob"
  ]
}

# variables.tf
variable "region" {
  description = "IBM Cloud region"
  type        = string
  default     = "us-south"
}

variable "resource_group_name" {
  description = "Name of the resource group"
  type        = string
  default     = "default"
}

# outputs.tf
output "vpc_id" {
  description = "ID of the created VPC"
  value       = ibm_is_vpc.production_vpc.id
}

output "vpc_crn" {
  description = "CRN of the created VPC"
  value       = ibm_is_vpc.production_vpc.crn
}
───────────────────────────────────────────────────────

This configuration:
✓ Creates a VPC in us-south region
✓ Uses proper naming conventions
✓ Includes resource tagging for management
✓ Provides outputs for reference in other configurations
✓ Uses variables for flexibility

Would you like me to apply this configuration? (yes/no)
```

### Example 2: VPC with Subnets

**Your Request:**
```bash
bob> Add 3 subnets to the VPC, one in each availability zone
```

**Bob's Response:**
```
I'll add 3 subnets across different availability zones. I'll update the 
configuration to include:

1. Data source to get available zones
2. Three subnet resources (one per zone)
3. Proper CIDR block allocation
4. Updated outputs

Updated configuration:
───────────────────────────────────────────────────────
# Add to main.tf

# Get available zones
data "ibm_is_zones" "regional" {
  region = var.region
}

# Subnet in zone 1
resource "ibm_is_subnet" "subnet_zone_1" {
  name                     = "${ibm_is_vpc.production_vpc.name}-subnet-1"
  vpc                      = ibm_is_vpc.production_vpc.id
  zone                     = data.ibm_is_zones.regional.zones[0]
  total_ipv4_address_count = 256
  resource_group           = data.ibm_resource_group.group.id
}

# Subnet in zone 2
resource "ibm_is_subnet" "subnet_zone_2" {
  name                     = "${ibm_is_vpc.production_vpc.name}-subnet-2"
  vpc                      = ibm_is_vpc.production_vpc.id
  zone                     = data.ibm_is_zones.regional.zones[1]
  total_ipv4_address_count = 256
  resource_group           = data.ibm_resource_group.group.id
}

# Subnet in zone 3
resource "ibm_is_subnet" "subnet_zone_3" {
  name                     = "${ibm_is_vpc.production_vpc.name}-subnet-3"
  vpc                      = ibm_is_vpc.production_vpc.id
  zone                     = data.ibm_is_zones.regional.zones[2]
  total_ipv4_address_count = 256
  resource_group           = data.ibm_resource_group.group.id
}

# Add to outputs.tf
output "subnet_ids" {
  description = "IDs of all created subnets"
  value = [
    ibm_is_subnet.subnet_zone_1.id,
    ibm_is_subnet.subnet_zone_2.id,
    ibm_is_subnet.subnet_zone_3.id
  ]
}
───────────────────────────────────────────────────────

Changes made:
✓ Added data source to dynamically get availability zones
✓ Created 3 subnets with 256 IPs each
✓ Distributed across different zones for high availability
✓ Used dynamic naming based on VPC name
✓ Added subnet IDs to outputs

Apply these changes? (yes/no)
```

---

## 🎯 Advanced Generation Techniques

### 1. Complex Infrastructure Requests

**Request:**
```bash
bob> Create a complete web application infrastructure with:
- VPC with public and private subnets
- Load balancer in public subnet
- Application servers in private subnet
- PostgreSQL database
- Object storage for static assets
- Security groups with proper rules
```

**Bob will generate:**
- Complete VPC architecture
- Load balancer configuration
- Auto-scaling group for app servers
- Managed PostgreSQL database
- Cloud Object Storage bucket
- Security groups with least-privilege rules
- All necessary networking components

### 2. Using Specifications

**Request:**
```bash
bob> Generate Terraform based on this specification:

Infrastructure Requirements:
- Environment: Production
- Region: us-south
- High Availability: Yes (3 zones)
- Compute: 6 VSIs (2 per zone)
- Database: PostgreSQL with read replicas
- Storage: 1TB object storage
- Network: Private connectivity only
- Security: Encryption at rest and in transit
```

**Bob will:**
- Parse the specification
- Generate appropriate resources
- Apply security best practices
- Configure high availability
- Set up monitoring and logging

### 3. Modifying Existing Code

**Request:**
```bash
bob> Review the current Terraform code and add:
- Encryption for all storage
- Private endpoints for services
- Network ACLs for subnets
- Logging to IBM Cloud Logs
```

**Bob will:**
- Analyze existing configuration
- Identify what needs to be added
- Generate the additional code
- Ensure compatibility with existing resources
- Update variables and outputs as needed

---

## 🔄 Iterative Refinement

### The Refinement Process

```bash
# Initial request
bob> Create a Kubernetes cluster

# Bob generates basic cluster

# Refine: Add features
bob> Add 3 worker pools with different instance types

# Refine: Enhance security
bob> Enable private service endpoint and disable public endpoint

# Refine: Add monitoring
bob> Configure logging to IBM Cloud Logs and monitoring to IBM Cloud Monitoring

# Refine: Optimize costs
bob> Use spot instances for non-critical worker pool

# Final review
bob> Review the complete configuration and suggest improvements
```

### Example Refinement Session

**Step 1: Initial Request**
```bash
bob> Create a VPC with a subnet
```

**Step 2: Add Security**
```bash
bob> Add a security group that only allows HTTPS traffic
```

**Step 3: Add Compute**
```bash
bob> Add 2 virtual server instances in the subnet
```

**Step 4: Add Load Balancer**
```bash
bob> Add a load balancer to distribute traffic between the instances
```

**Step 5: Optimize**
```bash
bob> Review the configuration and suggest cost optimizations
```

---

## 🛡️ Security-First Generation

### Bob's Security Practices

Bob automatically applies security best practices:

1. **Encryption by Default**
   - Enables encryption at rest for storage
   - Configures TLS for data in transit
   - Uses IBM Key Protect when appropriate

2. **Least Privilege Access**
   - Minimal security group rules
   - Private endpoints when possible
   - IAM policies with minimal permissions

3. **Network Isolation**
   - Private subnets for sensitive resources
   - Network ACLs for additional protection
   - VPN or Direct Link for hybrid connectivity

4. **Compliance**
   - Follows IBM Cloud best practices
   - Implements logging and monitoring
   - Enables audit trails

### Example: Secure Database

**Request:**
```bash
bob> Create a PostgreSQL database with maximum security
```

**Bob generates:**
```hcl
# Secure PostgreSQL configuration
resource "ibm_database" "postgresql" {
  name              = "secure-postgresql"
  plan              = "standard"
  location          = var.region
  service           = "databases-for-postgresql"
  resource_group_id = data.ibm_resource_group.group.id
  
  # High availability
  members_memory_allocation_mb = 4096
  members_disk_allocation_mb   = 20480
  members_cpu_allocation_count = 3
  
  # Security configurations
  service_endpoints = "private"  # Private endpoint only
  
  # Encryption
  key_protect_key = ibm_kms_key.database_key.id
  
  # Backup configuration
  backup_id = ibm_database_backup.postgresql_backup.backup_id
  
  # Network isolation
  allowlist {
    address     = ibm_is_subnet.private_subnet.ipv4_cidr_block
    description = "Allow from private subnet only"
  }
  
  tags = [
    "security:high",
    "compliance:required",
    "encryption:enabled"
  ]
}

# Key Protect for encryption
resource "ibm_kms_key" "database_key" {
  instance_id  = ibm_resource_instance.key_protect.guid
  key_name     = "postgresql-encryption-key"
  standard_key = false
}
```

---

## 📊 Real-World Examples

### Example 1: E-Commerce Platform

**Request:**
```bash
bob> Create infrastructure for an e-commerce platform with:
- Multi-zone deployment for high availability
- Separate environments (dev, staging, prod)
- Microservices architecture
- Redis for caching
- PostgreSQL for transactions
- Object storage for product images
- CDN for static content
```

**Bob generates:**
- Complete VPC architecture per environment
- Kubernetes clusters for microservices
- Managed databases with replication
- Redis cluster for caching
- COS buckets with CDN integration
- Security groups and network policies
- Monitoring and logging setup

### Example 2: Data Analytics Platform

**Request:**
```bash
bob> Set up a data analytics platform with:
- Data lake using Object Storage
- Apache Spark cluster for processing
- PostgreSQL for metadata
- Jupyter notebooks for analysis
- Private network connectivity
- Data encryption throughout
```

**Bob generates:**
- COS buckets with lifecycle policies
- Analytics Engine (Spark) cluster
- Database for metadata management
- Code Engine for Jupyter notebooks
- VPN gateway for secure access
- Encryption keys and policies

### Example 3: CI/CD Pipeline Infrastructure

**Request:**
```bash
bob> Create CI/CD infrastructure with:
- Container registry
- Kubernetes cluster for deployments
- Object storage for artifacts
- PostgreSQL for pipeline metadata
- Monitoring and logging
```

**Bob generates:**
- IBM Container Registry setup
- IKS cluster with proper sizing
- COS buckets for build artifacts
- Database for pipeline state
- Cloud Logs and Monitoring integration
- Service accounts and RBAC

---

## 🎓 Best Practices for AI-Assisted Development

### 1. Be Specific in Your Requests

**❌ Vague:**
```bash
bob> Create some infrastructure
```

**✅ Specific:**
```bash
bob> Create a VPC in us-south with 3 subnets across different zones, 
each subnet should have 256 IP addresses
```

### 2. Provide Context

**❌ No Context:**
```bash
bob> Add a database
```

**✅ With Context:**
```bash
bob> Add a PostgreSQL database for the production environment with 
high availability, 16GB RAM, and private endpoint only
```

### 3. Iterate Incrementally

**✅ Good Approach:**
```bash
# Step 1
bob> Create a basic VPC

# Step 2
bob> Add subnets to the VPC

# Step 3
bob> Add security groups

# Step 4
bob> Add compute instances
```

### 4. Review and Understand

**Always:**
- Read the generated code
- Understand what each resource does
- Ask Bob to explain unclear parts
- Verify security configurations

```bash
bob> Explain why you chose this security group configuration

bob> What are the cost implications of this setup?

bob> Are there any security concerns I should be aware of?
```

### 5. Use Bob for Code Review

```bash
bob> Review this Terraform code and suggest improvements:
[paste your code]

bob> Check this configuration for security issues

bob> Optimize this code for cost efficiency
```

---

## 🔍 Code Review with Bob

### Automated Review

**Request:**
```bash
bob> Review my current Terraform configuration and provide feedback
```

**Bob analyzes:**
- Security vulnerabilities
- Cost optimization opportunities
- Best practice violations
- Resource dependencies
- Naming conventions
- Documentation completeness

### Example Review Session

```bash
bob> Review this VPC configuration:

resource "ibm_is_vpc" "vpc" {
  name = "vpc1"
}

resource "ibm_is_subnet" "subnet" {
  name = "subnet1"
  vpc = ibm_is_vpc.vpc.id
  zone = "us-south-1"
  ipv4_cidr_block = "10.0.0.0/24"
}
```

**Bob's Response:**
```
I've reviewed your VPC configuration. Here are my findings:

🔴 Issues Found:
1. Missing resource group assignment
2. No tags for resource management
3. Generic naming (vpc1, subnet1)
4. No security group defined
5. Missing outputs for resource IDs

🟡 Suggestions:
1. Add descriptive names with environment prefix
2. Include resource tagging strategy
3. Define security groups before instances
4. Add outputs for integration with other configs
5. Consider using variables for flexibility

🟢 Improvements:
Here's the improved version:

[Bob provides improved code with all fixes]

Would you like me to apply these improvements?
```

---

## 💡 Advanced Tips and Tricks

### 1. Using Templates

```bash
bob> Use the standard VPC template and customize it for production 
with 5 subnets and enhanced security
```

### 2. Generating Modules

```bash
bob> Create a reusable Terraform module for a standard 3-tier 
application architecture
```

### 3. Multi-Environment Setup

```bash
bob> Generate Terraform workspaces for dev, staging, and production 
environments with appropriate sizing for each
```

### 4. Cost Estimation

```bash
bob> Estimate the monthly cost of this infrastructure

bob> Suggest cost optimizations for this configuration
```

### 5. Documentation Generation

```bash
bob> Generate comprehensive documentation for this Terraform code 
including architecture diagrams and usage instructions
```

---

## 🐛 Troubleshooting AI Generation

### Issue 1: Generated Code Doesn't Apply

**Solution:**
```bash
bob> The terraform apply failed with this error: [paste error]
Please fix the configuration
```

### Issue 2: Code Doesn't Match Requirements

**Solution:**
```bash
bob> The generated code creates 2 subnets but I need 3. 
Please update the configuration
```

### Issue 3: Security Concerns

**Solution:**
```bash
bob> Review the security configuration and make it more restrictive. 
Only allow traffic from specific IP ranges
```

### Issue 4: Performance Issues

**Solution:**
```bash
bob> This configuration seems over-provisioned. 
Optimize it for a small development environment
```

---

## 📈 Measuring Success

### Productivity Gains

**Traditional Approach:**
- 2-4 hours to write VPC configuration
- 1-2 hours for security groups
- 30 minutes for documentation
- **Total: 3.5-6.5 hours**

**With Bob:**
- 5 minutes to describe requirements
- 2 minutes to review generated code
- 1 minute to apply
- **Total: 8 minutes**

**Result: 95% time savings**

### Quality Improvements

- ✅ Consistent code style
- ✅ Best practices applied automatically
- ✅ Security by default
- ✅ Complete documentation
- ✅ Fewer errors and bugs

---

## 🎯 Practice Exercises

### Exercise 1: Basic Generation
```bash
bob> Create a VPC with 2 subnets and a security group that allows 
HTTP and HTTPS traffic
```

### Exercise 2: Iterative Refinement
```bash
# Start simple
bob> Create a Kubernetes cluster

# Add features
bob> Add 2 worker pools with different instance types
bob> Enable private service endpoint
bob> Add monitoring and logging
```

### Exercise 3: Code Review
```bash
# Paste your existing Terraform code
bob> Review this code and suggest improvements:
[your code here]
```

### Exercise 4: Complex Infrastructure
```bash
bob> Create a complete 3-tier web application infrastructure with 
load balancer, application servers, and database, all with high 
availability and security best practices
```

---

## 🎓 Next Steps

Now that you've mastered AI-assisted Terraform generation:

1. **[TIM MCP Integration](./04-tim-mcp-integration.md)** - Connect to IBM Cloud services
2. **[Best Practices](./05-best-practices.md)** - Learn advanced patterns
3. **[IAC-Spec-Kit](./06-iac-spec-kit.md)** - Create detailed specifications

---

## 📚 Additional Resources

- [Bob-Shell Documentation](https://bob-shell.dev/docs)
- [Terraform Best Practices](https://www.terraform-best-practices.com/)
- [IBM Cloud Architecture Patterns](https://www.ibm.com/cloud/architecture)

---

**Ready to integrate with IBM Cloud services?** Continue to [TIM MCP Integration](./04-tim-mcp-integration.md) →
