# Terraform Basics for IBM Cloud

> **Master Infrastructure as Code fundamentals for IBM Cloud**

## 📋 Overview

This guide covers Terraform essentials specifically for IBM Cloud, helping you understand how to define, provision, and manage cloud infrastructure using code.

**What You'll Learn:**
- Infrastructure as Code (IaC) concepts
- Terraform core components and workflow
- IBM Cloud Provider configuration
- Essential Terraform commands
- State management best practices
- Common IBM Cloud resource patterns

**Prerequisites:**
- Completed [Getting Started Guide](./01-getting-started.md)
- IBM Cloud account with API key configured
- Basic understanding of cloud infrastructure concepts

**Estimated Time:** 45-60 minutes

---

## 🎯 What is Infrastructure as Code?

### Traditional Infrastructure Management

```
Manual Process:
1. Log into IBM Cloud Console
2. Click through UI to create resources
3. Configure settings manually
4. Repeat for each environment
5. Document changes (maybe)
6. Hope nothing breaks

Problems:
❌ Time-consuming and error-prone
❌ Difficult to replicate
❌ No version control
❌ Hard to audit changes
❌ Inconsistent across environments
```

### Infrastructure as Code Approach

```
IaC Process:
1. Write infrastructure configuration in code
2. Version control your infrastructure
3. Review changes like code reviews
4. Apply changes automatically
5. Replicate across environments easily
6. Audit trail built-in

Benefits:
✅ Repeatable and consistent
✅ Version controlled
✅ Automated deployment
✅ Easy to review and audit
✅ Self-documenting
✅ Testable infrastructure
```

---

## 🏗️ Terraform Core Concepts

### 1. Providers

Providers are plugins that interact with cloud platforms, SaaS providers, and APIs.

```hcl
# IBM Cloud Provider
terraform {
  required_providers {
    ibm = {
      source  = "IBM-Cloud/ibm"
      version = "~> 1.50.0"
    }
  }
}

provider "ibm" {
  ibmcloud_api_key = var.ibmcloud_api_key
  region           = var.region
}
```

### 2. Resources

Resources are the most important element - they describe infrastructure objects.

```hcl
# A VPC resource
resource "ibm_is_vpc" "example" {
  name = "my-vpc"
  resource_group = var.resource_group_id
}

# A subnet resource
resource "ibm_is_subnet" "example" {
  name            = "my-subnet"
  vpc             = ibm_is_vpc.example.id
  zone            = "us-south-1"
  ipv4_cidr_block = "10.240.0.0/24"
}
```

### 3. Variables

Variables make your configurations flexible and reusable.

```hcl
# Input variables
variable "region" {
  description = "IBM Cloud region"
  type        = string
  default     = "us-south"
}

variable "vpc_name" {
  description = "Name of the VPC"
  type        = string
  validation {
    condition     = length(var.vpc_name) > 0
    error_message = "VPC name cannot be empty"
  }
}
```

### 4. Outputs

Outputs expose information about your infrastructure.

```hcl
output "vpc_id" {
  description = "ID of the created VPC"
  value       = ibm_is_vpc.example.id
}

output "subnet_id" {
  description = "ID of the created subnet"
  value       = ibm_is_subnet.example.id
}
```

### 5. Data Sources

Data sources query existing infrastructure.

```hcl
# Get existing resource group
data "ibm_resource_group" "group" {
  name = "default"
}

# Get available zones
data "ibm_is_zones" "regional" {
  region = var.region
}
```

---

## 🔧 IBM Cloud Provider Configuration

### Basic Configuration

```hcl
# versions.tf
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    ibm = {
      source  = "IBM-Cloud/ibm"
      version = "~> 1.50.0"
    }
  }
}

# provider.tf
provider "ibm" {
  ibmcloud_api_key = var.ibmcloud_api_key
  region           = var.region
}
```

### Authentication Methods

#### Method 1: Environment Variables (Recommended)

```bash
export IBMCLOUD_API_KEY="your-api-key"
export IC_REGION="us-south"
```

```hcl
provider "ibm" {
  # Automatically uses IBMCLOUD_API_KEY from environment
  region = var.region
}
```

#### Method 2: Variables File

```hcl
# variables.tf
variable "ibmcloud_api_key" {
  description = "IBM Cloud API Key"
  type        = string
  sensitive   = true
}

# terraform.tfvars (DO NOT commit to git!)
ibmcloud_api_key = "your-api-key"
```

#### Method 3: Using Bob-Shell

```bash
bob

bob> configure ibm-cloud credentials
# Bob will securely manage your credentials
```

### Regional Configuration

```hcl
# Multi-region setup
provider "ibm" {
  alias            = "us_south"
  ibmcloud_api_key = var.ibmcloud_api_key
  region           = "us-south"
}

provider "ibm" {
  alias            = "eu_de"
  ibmcloud_api_key = var.ibmcloud_api_key
  region           = "eu-de"
}

# Use specific provider
resource "ibm_is_vpc" "us_vpc" {
  provider = ibm.us_south
  name     = "us-vpc"
}
```

---

## 📝 Essential Terraform Commands

### Initialize Terraform

```bash
# Initialize working directory
terraform init

# Upgrade providers
terraform init -upgrade

# Reconfigure backend
terraform init -reconfigure
```

**What it does:**
- Downloads required providers
- Initializes backend for state storage
- Prepares working directory

### Format and Validate

```bash
# Format code to canonical style
terraform fmt

# Format and show differences
terraform fmt -diff

# Validate configuration
terraform validate
```

### Plan Changes

```bash
# Preview changes
terraform plan

# Save plan to file
terraform plan -out=tfplan

# Plan for specific resource
terraform plan -target=ibm_is_vpc.example
```

**Understanding Plan Output:**
```
+ create    # New resource will be created
~ update    # Existing resource will be modified
- destroy   # Resource will be destroyed
-/+ replace # Resource will be destroyed and recreated
```

### Apply Changes

```bash
# Apply changes (with confirmation)
terraform apply

# Apply saved plan
terraform apply tfplan

# Auto-approve (use with caution!)
terraform apply -auto-approve

# Apply specific resource
terraform apply -target=ibm_is_vpc.example
```

### Inspect State

```bash
# List resources in state
terraform state list

# Show resource details
terraform state show ibm_is_vpc.example

# Show all outputs
terraform output

# Show specific output
terraform output vpc_id
```

### Destroy Resources

```bash
# Destroy all resources (with confirmation)
terraform destroy

# Destroy specific resource
terraform destroy -target=ibm_is_vpc.example

# Auto-approve destruction (dangerous!)
terraform destroy -auto-approve
```

---

## 🗂️ Project Structure Best Practices

### Basic Structure

```
my-ibm-cloud-project/
├── main.tf              # Main resource definitions
├── variables.tf         # Input variable declarations
├── outputs.tf           # Output value declarations
├── versions.tf          # Provider version constraints
├── terraform.tfvars     # Variable values (DO NOT commit!)
├── .gitignore          # Ignore sensitive files
└── README.md           # Project documentation
```

### Advanced Structure

```
my-ibm-cloud-project/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   │   └── ...
│   └── production/
│       └── ...
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── kubernetes/
│   │   └── ...
│   └── database/
│       └── ...
├── .gitignore
└── README.md
```

### Essential .gitignore

```gitignore
# Terraform files
*.tfstate
*.tfstate.*
.terraform/
.terraform.lock.hcl
crash.log
override.tf
override.tf.json

# Sensitive files
terraform.tfvars
*.tfvars
*.auto.tfvars
.env

# OS files
.DS_Store
Thumbs.db
```

---

## 💾 State Management

### What is Terraform State?

Terraform state is a JSON file that maps your configuration to real-world resources.

```json
{
  "version": 4,
  "terraform_version": "1.6.0",
  "resources": [
    {
      "type": "ibm_is_vpc",
      "name": "example",
      "instances": [
        {
          "attributes": {
            "id": "r006-abc123...",
            "name": "my-vpc",
            "status": "available"
          }
        }
      ]
    }
  ]
}
```

### Local State (Default)

```hcl
# State stored in local file: terraform.tfstate
# Good for: Learning, testing, single-user projects
# Bad for: Teams, production, CI/CD
```

### Remote State (Recommended)

#### Using IBM Cloud Object Storage

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket                      = "my-terraform-state"
    key                         = "terraform.tfstate"
    region                      = "us-south"
    endpoint                    = "s3.us-south.cloud-object-storage.appdomain.cloud"
    skip_credentials_validation = true
    skip_region_validation      = true
  }
}
```

#### Using Terraform Cloud

```hcl
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      name = "ibm-cloud-prod"
    }
  }
}
```

### State Commands

```bash
# List resources
terraform state list

# Show resource
terraform state show ibm_is_vpc.example

# Move resource
terraform state mv ibm_is_vpc.old ibm_is_vpc.new

# Remove resource from state
terraform state rm ibm_is_vpc.example

# Pull remote state
terraform state pull

# Push local state
terraform state push
```

---

## 🎯 Common IBM Cloud Resources

### 1. Resource Group

```hcl
resource "ibm_resource_group" "group" {
  name = "my-resource-group"
}
```

### 2. VPC

```hcl
resource "ibm_is_vpc" "vpc" {
  name           = "my-vpc"
  resource_group = ibm_resource_group.group.id
  
  tags = [
    "env:production",
    "managed-by:terraform"
  ]
}
```

### 3. Subnet

```hcl
resource "ibm_is_subnet" "subnet" {
  name                     = "my-subnet"
  vpc                      = ibm_is_vpc.vpc.id
  zone                     = "us-south-1"
  total_ipv4_address_count = 256
  resource_group           = ibm_resource_group.group.id
}
```

### 4. Security Group

```hcl
resource "ibm_is_security_group" "sg" {
  name           = "my-security-group"
  vpc            = ibm_is_vpc.vpc.id
  resource_group = ibm_resource_group.group.id
}

resource "ibm_is_security_group_rule" "allow_ssh" {
  group     = ibm_is_security_group.sg.id
  direction = "inbound"
  remote    = "0.0.0.0/0"
  
  tcp {
    port_min = 22
    port_max = 22
  }
}
```

### 5. Virtual Server Instance

```hcl
resource "ibm_is_instance" "vsi" {
  name    = "my-vsi"
  vpc     = ibm_is_vpc.vpc.id
  zone    = "us-south-1"
  profile = "cx2-2x4"
  
  primary_network_interface {
    subnet          = ibm_is_subnet.subnet.id
    security_groups = [ibm_is_security_group.sg.id]
  }
  
  boot_volume {
    name = "my-boot-volume"
  }
  
  image = data.ibm_is_image.ubuntu.id
  keys  = [ibm_is_ssh_key.key.id]
}

data "ibm_is_image" "ubuntu" {
  name = "ibm-ubuntu-20-04-minimal-amd64-2"
}
```

### 6. Kubernetes Cluster

```hcl
resource "ibm_container_vpc_cluster" "cluster" {
  name              = "my-cluster"
  vpc_id            = ibm_is_vpc.vpc.id
  flavor            = "bx2.4x16"
  worker_count      = 3
  resource_group_id = ibm_resource_group.group.id
  
  zones {
    subnet_id = ibm_is_subnet.subnet.id
    name      = "us-south-1"
  }
}
```

### 7. Cloud Object Storage

```hcl
resource "ibm_resource_instance" "cos" {
  name              = "my-cos-instance"
  service           = "cloud-object-storage"
  plan              = "standard"
  location          = "global"
  resource_group_id = ibm_resource_group.group.id
}

resource "ibm_cos_bucket" "bucket" {
  bucket_name          = "my-unique-bucket-name"
  resource_instance_id = ibm_resource_instance.cos.id
  region_location      = "us-south"
  storage_class        = "standard"
}
```

---

## 🔄 Terraform Workflow

### Development Workflow

```bash
# 1. Write configuration
vim main.tf

# 2. Format code
terraform fmt

# 3. Validate syntax
terraform validate

# 4. Preview changes
terraform plan

# 5. Apply changes
terraform apply

# 6. Verify outputs
terraform output
```

### Using Bob-Shell

```bash
bob

# Bob handles the workflow for you
bob> Create a VPC with 3 subnets across different zones

# Bob will:
# 1. Generate Terraform code
# 2. Format and validate
# 3. Show you the plan
# 4. Apply after confirmation
# 5. Display outputs
```

---

## 🎓 Hands-On Exercise

### Exercise 1: Create a Basic VPC

**Goal:** Create a VPC with a subnet and security group.

```hcl
# main.tf
terraform {
  required_providers {
    ibm = {
      source  = "IBM-Cloud/ibm"
      version = "~> 1.50.0"
    }
  }
}

provider "ibm" {
  region = "us-south"
}

# Variables
variable "vpc_name" {
  default = "exercise-vpc"
}

# Resource Group
data "ibm_resource_group" "group" {
  name = "default"
}

# VPC
resource "ibm_is_vpc" "vpc" {
  name           = var.vpc_name
  resource_group = data.ibm_resource_group.group.id
}

# Subnet
resource "ibm_is_subnet" "subnet" {
  name                     = "${var.vpc_name}-subnet"
  vpc                      = ibm_is_vpc.vpc.id
  zone                     = "us-south-1"
  total_ipv4_address_count = 256
}

# Security Group
resource "ibm_is_security_group" "sg" {
  name = "${var.vpc_name}-sg"
  vpc  = ibm_is_vpc.vpc.id
}

# Outputs
output "vpc_id" {
  value = ibm_is_vpc.vpc.id
}

output "subnet_id" {
  value = ibm_is_subnet.subnet.id
}
```

**Steps:**

```bash
# 1. Initialize
terraform init

# 2. Plan
terraform plan

# 3. Apply
terraform apply

# 4. Verify
terraform output

# 5. Clean up
terraform destroy
```

### Exercise 2: Use Bob-Shell

```bash
bob

# Try these commands:
bob> Create a VPC named "learning-vpc" in us-south

bob> Add a public subnet to the VPC in zone us-south-1

bob> Create a security group that allows SSH and HTTPS

bob> Show me the current infrastructure

bob> Destroy all resources
```

---

## 🐛 Common Issues and Solutions

### Issue 1: Provider Authentication Failed

```
Error: Error authenticating with IBM Cloud
```

**Solution:**
```bash
# Verify API key is set
echo $IBMCLOUD_API_KEY

# Test authentication
ibmcloud login --apikey $IBMCLOUD_API_KEY

# Check IAM permissions
ibmcloud iam user-policies $(ibmcloud account user-preference --output json | jq -r .email)
```

### Issue 2: Resource Already Exists

```
Error: Resource already exists
```

**Solution:**
```bash
# Import existing resource
terraform import ibm_is_vpc.vpc r006-abc123...

# Or use different name
```

### Issue 3: State Lock

```
Error: Error acquiring the state lock
```

**Solution:**
```bash
# Force unlock (use with caution!)
terraform force-unlock <lock-id>
```

### Issue 4: Quota Exceeded

```
Error: Quota exceeded for resource type
```

**Solution:**
- Check IBM Cloud quotas in console
- Request quota increase
- Clean up unused resources

---

## 📚 Best Practices Summary

1. **Always use version control** for Terraform code
2. **Never commit sensitive data** (API keys, passwords)
3. **Use remote state** for team collaboration
4. **Format and validate** before committing
5. **Review plans carefully** before applying
6. **Use modules** for reusable components
7. **Tag resources** for organization and cost tracking
8. **Document your code** with comments and README
9. **Test in dev** before applying to production
10. **Use Bob-Shell** to accelerate development

---

## 🎯 Next Steps

Now that you understand Terraform basics, you're ready to:

1. **[AI-Assisted Terraform Generation](./03-bob-terraform-generation.md)** - Let Bob write Terraform for you
2. **[TIM MCP Integration](./04-tim-mcp-integration.md)** - Connect to IBM Cloud services
3. **[Examples](../examples/)** - Try real-world scenarios

---

## 📖 Additional Resources

- [Terraform Documentation](https://www.terraform.io/docs)
- [IBM Cloud Provider Docs](https://registry.terraform.io/providers/IBM-Cloud/ibm/latest/docs)
- [Terraform Best Practices](https://www.terraform-best-practices.com/)
- [IBM Cloud Architecture Center](https://www.ibm.com/cloud/architecture)

---

**Ready to let AI write your Terraform code?** Continue to [AI-Assisted Terraform Generation](./03-bob-terraform-generation.md) →
