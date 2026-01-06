# Getting Started with Bob IDE & Bob-Shell for IBM Cloud

> **Your journey to AI-powered IBM Cloud infrastructure management starts here**

## 📋 Prerequisites

Before you begin, ensure you have:

- **Operating System**: macOS, Linux, or Windows (WSL2)
- **IBM Cloud Account**: [Sign up for free](https://cloud.ibm.com/registration)
- **IBM Cloud API Key**: [Create one here](https://cloud.ibm.com/iam/apikeys)
- **Basic Command Line Knowledge**: Familiarity with terminal/shell commands
- **Internet Connection**: For downloading tools and accessing IBM Cloud

## 🎯 What You'll Accomplish

By the end of this guide, you will:
- ✅ Install Bob IDE and Bob-Shell
- ✅ Configure IBM Cloud credentials
- ✅ Set up your first workspace
- ✅ Execute your first Bob-Shell command
- ✅ Deploy a simple IBM Cloud resource

**Estimated Time**: 15-20 minutes

---

## Step 1: Install Bob IDE

### macOS

```bash
# Using Homebrew (recommended)
brew install bob-ide

# Or using the install script
curl -fsSL https://bob-ide.com/install.sh | sh
```

### Linux

```bash
# Using the install script
curl -fsSL https://bob-ide.com/install.sh | sh

# Or download the binary directly
wget https://bob-ide.com/downloads/bob-ide-linux-amd64
chmod +x bob-ide-linux-amd64
sudo mv bob-ide-linux-amd64 /usr/local/bin/bob-ide
```

### Windows (WSL2)

```bash
# Inside WSL2 terminal
curl -fsSL https://bob-ide.com/install.sh | sh
```

### Verify Installation

```bash
bob-ide --version
# Expected output: Bob IDE v1.x.x
```

---

## Step 2: Install Bob-Shell

Bob-Shell is the command-line interface that brings AI-powered assistance to your terminal.

### Installation

```bash
# macOS/Linux
curl -fsSL https://bob-shell.dev/install.sh | sh

# Add to PATH (if not automatically added)
echo 'export PATH="$HOME/.bob-shell/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# For zsh users
echo 'export PATH="$HOME/.bob-shell/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### Verify Installation

```bash
bob --version
# Expected output: Bob-Shell v1.x.x
```

### First Launch

```bash
bob
```

You should see the Bob-Shell welcome screen:

```
╔══════════════════════════════════════════════════════════╗
║                    Welcome to Bob-Shell                   ║
║              Your AI-Powered Cloud Assistant              ║
╚══════════════════════════════════════════════════════════╝

Type 'help' for available commands or just ask me anything!

bob>
```

---

## Step 3: Configure IBM Cloud Credentials

### Create an IBM Cloud API Key

1. Log in to [IBM Cloud Console](https://cloud.ibm.com)
2. Navigate to **Manage** → **Access (IAM)** → **API keys**
3. Click **Create an IBM Cloud API key**
4. Give it a descriptive name (e.g., "Bob-Shell-Dev")
5. Click **Create** and **Download** or copy the key immediately
   - ⚠️ **Important**: You won't be able to see this key again!

### Configure Credentials in Bob-Shell

#### Method 1: Environment Variables (Recommended)

```bash
# Add to your shell profile (~/.bashrc or ~/.zshrc)
export IBMCLOUD_API_KEY="your-api-key-here"
export IC_REGION="us-south"  # Optional: default region
export IC_RESOURCE_GROUP="default"  # Optional: default resource group

# Reload your shell
source ~/.bashrc  # or source ~/.zshrc
```

#### Method 2: Bob-Shell Configuration File

```bash
# Create Bob-Shell config directory
mkdir -p ~/.bob-shell

# Create configuration file
cat > ~/.bob-shell/config.yaml << EOF
ibm_cloud:
  api_key: "your-api-key-here"
  region: "us-south"
  resource_group: "default"
EOF

# Secure the config file
chmod 600 ~/.bob-shell/config.yaml
```

#### Method 3: Interactive Setup

```bash
bob config setup
```

Follow the interactive prompts to configure your IBM Cloud credentials.

### Verify Configuration

```bash
bob

# Inside Bob-Shell
bob> verify ibm-cloud connection
```

Expected output:
```
✓ IBM Cloud API Key: Valid
✓ Region: us-south
✓ Resource Group: default
✓ Connection: Successful
```

---

## Step 4: Set Up Your First Workspace

### Create a Project Directory

```bash
# Create a directory for your IBM Cloud projects
mkdir -p ~/ibm-cloud-projects/my-first-project
cd ~/ibm-cloud-projects/my-first-project

# Initialize the workspace
bob init
```

### Workspace Structure

Bob-Shell will create the following structure:

```
my-first-project/
├── .bob/
│   ├── config.yaml          # Project-specific configuration
│   ├── state/               # Local state files
│   └── cache/               # Cached resources
├── terraform/               # Terraform configurations
├── scripts/                 # Helper scripts
└── README.md               # Project documentation
```

### Configure Project Settings

Edit `.bob/config.yaml`:

```yaml
project:
  name: "my-first-project"
  description: "My first IBM Cloud project with Bob"
  
ibm_cloud:
  region: "us-south"
  resource_group: "default"
  
terraform:
  version: "1.6.0"
  backend: "local"  # or "ibm-cos" for remote state
```

---

## Step 5: Your First Bob-Shell Command

Let's create a simple IBM Cloud resource using Bob's AI assistance.

### Example 1: Create a Resource Group

```bash
bob

# Inside Bob-Shell
bob> Create a resource group named "dev-environment" in IBM Cloud
```

Bob will:
1. 🤔 Understand your request
2. 📝 Generate the appropriate Terraform code
3. 💡 Explain what it's doing
4. ✅ Ask for confirmation before applying

**Bob's Response:**
```
I'll help you create a resource group in IBM Cloud. Here's what I'll do:

1. Generate Terraform configuration for the resource group
2. Initialize Terraform
3. Apply the configuration

Terraform code:
───────────────────────────────────────────────────────
terraform {
  required_providers {
    ibm = {
      source  = "IBM-Cloud/ibm"
      version = "~> 1.50"
    }
  }
}

provider "ibm" {
  ibmcloud_api_key = var.ibmcloud_api_key
  region           = "us-south"
}

resource "ibm_resource_group" "dev_environment" {
  name = "dev-environment"
}

output "resource_group_id" {
  value = ibm_resource_group.dev_environment.id
}
───────────────────────────────────────────────────────

Proceed with creation? (yes/no):
```

Type `yes` to proceed.

### Example 2: Create a VPC

```bash
bob> Create a VPC named "my-vpc" with a public subnet in us-south-1
```

Bob will generate a complete VPC configuration with:
- VPC resource
- Public subnet
- Public gateway
- Security group with basic rules

### Example 3: Ask Questions

```bash
bob> What are the available IBM Cloud regions?

bob> How do I create a Kubernetes cluster?

bob> Show me best practices for VPC security groups
```

---

## Step 6: Understanding Bob's Workflow

### The Bob-Shell Process

```
┌─────────────────────────────────────────────────────────┐
│  1. You describe what you want in natural language      │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│  2. Bob analyzes your request and IBM Cloud context     │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│  3. Bob generates appropriate Terraform/CLI code        │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│  4. Bob explains the code and asks for confirmation     │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│  5. Bob executes and provides feedback                  │
└─────────────────────────────────────────────────────────┘
```

### Key Features

- **Context-Aware**: Bob remembers your conversation and project context
- **Best Practices**: Bob suggests secure and efficient configurations
- **Iterative**: Easily refine and modify generated code
- **Educational**: Bob explains what it's doing and why

---

## Step 7: Essential Bob-Shell Commands

### Navigation and Help

```bash
# Get help
bob> help

# List available commands
bob> commands

# Get help for a specific topic
bob> help terraform
bob> help ibm-cloud
```

### Project Management

```bash
# Show current project status
bob> status

# List resources in current project
bob> list resources

# Show project configuration
bob> show config
```

### Terraform Operations

```bash
# Initialize Terraform
bob> terraform init

# Plan changes
bob> terraform plan

# Apply changes
bob> terraform apply

# Destroy resources
bob> terraform destroy
```

### IBM Cloud Operations

```bash
# List IBM Cloud resources
bob> list vpcs
bob> list clusters
bob> list databases

# Get resource details
bob> describe vpc my-vpc
bob> show cluster my-cluster
```

---

## Step 8: Best Practices for Getting Started

### 1. Start Small
Begin with simple resources like resource groups or VPCs before moving to complex architectures.

### 2. Use Descriptive Names
```bash
# Good
bob> Create a VPC named "production-vpc-us-south"

# Less clear
bob> Create a VPC named "vpc1"
```

### 3. Review Before Applying
Always review the generated Terraform code before applying changes.

### 4. Use Version Control
```bash
cd ~/ibm-cloud-projects/my-first-project
git init
git add .
git commit -m "Initial project setup"
```

### 5. Organize Your Projects
```
~/ibm-cloud-projects/
├── dev-environment/
├── staging-environment/
└── production-environment/
```

### 6. Keep Credentials Secure
- Never commit API keys to version control
- Use environment variables or secure vaults
- Rotate API keys regularly

---

## 🎓 Next Steps

Congratulations! You've successfully set up Bob IDE and Bob-Shell for IBM Cloud. Here's what to explore next:

1. **[Terraform Basics](./02-terraform-basics.md)** - Learn Infrastructure as Code fundamentals
2. **[AI-Assisted Generation](./03-bob-terraform-generation.md)** - Master Bob's code generation
3. **[Examples](../examples/)** - Try real-world scenarios

---

## 🐛 Troubleshooting

### Bob-Shell Won't Start

```bash
# Check if Bob-Shell is in PATH
which bob

# Reinstall if necessary
curl -fsSL https://bob-shell.dev/install.sh | sh
```

### IBM Cloud Authentication Fails

```bash
# Verify API key is set
echo $IBMCLOUD_API_KEY

# Test API key manually
ibmcloud login --apikey $IBMCLOUD_API_KEY

# Check IAM permissions
ibmcloud iam user-policies <your-email>
```

### Terraform Errors

```bash
# Clear Terraform cache
rm -rf .terraform
bob> terraform init

# Check Terraform version
terraform version
```

### Bob Can't Find Resources

```bash
# Refresh Bob's cache
bob> refresh cache

# Verify region and resource group
bob> show config
```

---

## 📚 Additional Resources

- [Bob-Shell Documentation](https://bob-shell.dev/docs)
- [IBM Cloud CLI Reference](https://cloud.ibm.com/docs/cli)
- [Terraform IBM Provider Docs](https://registry.terraform.io/providers/IBM-Cloud/ibm/latest/docs)
- [IBM Cloud Architecture Center](https://www.ibm.com/cloud/architecture)

---

## 💬 Get Help

- **Community Forum**: [forum.bob-ide.com](https://forum.bob-ide.com)
- **Discord**: [discord.gg/bob-ide](https://discord.gg/bob-ide)
- **GitHub Issues**: [github.com/bob-ide/bob-shell](https://github.com/bob-ide/bob-shell)

---

**Ready to dive deeper?** Continue to [Terraform Basics for IBM Cloud](./02-terraform-basics.md) →
