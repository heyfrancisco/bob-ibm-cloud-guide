# IBM Cloud MCP Server Integration with Bob-Shell

> **Connect Bob-Shell to IBM Cloud services using the Model Context Protocol**

## 📋 Overview

The IBM Cloud MCP Server enables Bob-Shell to directly interact with IBM Cloud services through the Model Context Protocol (MCP). Built into the IBM Cloud CLI, it provides real-time access to your IBM Cloud resources, enabling intelligent infrastructure management and automated operations.

**What You'll Learn:**
- Understanding MCP and IBM Cloud MCP Server
- Installing and configuring IBM Cloud MCP Server globally
- Connecting Bob-Shell to IBM Cloud services
- Managing resources through MCP
- Advanced integration scenarios
- Security best practices

**Prerequisites:**
- Bob-Shell installed and configured
- IBM Cloud account with appropriate permissions
- Basic command line knowledge

**Estimated Time:** 20-30 minutes

---

## 🔌 What is MCP?

### Model Context Protocol (MCP)

MCP is an open protocol that enables AI assistants to securely connect to external data sources and tools. It provides:

- **Standardized Communication**: Consistent way for AI to interact with services
- **Secure Access**: Controlled permissions and authentication
- **Real-time Data**: Live access to cloud resources and state
- **Tool Integration**: Execute operations directly from AI conversations

### IBM Cloud MCP Server

The IBM Cloud MCP Server is built directly into the IBM Cloud CLI and provides:

- Direct connection to IBM Cloud APIs
- Real-time resource information
- Secure, authenticated access
- Read-only by default (safe mode)
- Optional write operations with explicit flag
- Support for all IBM Cloud CLI commands

### Architecture

```
┌─────────────────────────────────────────────────────────┐
│                      Bob-Shell                          │
│                   (MCP Client)                          │
└────────────────────┬────────────────────────────────────┘
                     │ MCP Protocol (stdio)
                     ▼
┌─────────────────────────────────────────────────────────┐
│              IBM Cloud CLI (MCP Server)                 │
│              Built-in MCP Support                       │
└────────────────────┬────────────────────────────────────┘
                     │ IBM Cloud APIs
                     ▼
┌─────────────────────────────────────────────────────────┐
│                    IBM Cloud                            │
│         (VPC, Kubernetes, Databases, etc.)              │
└─────────────────────────────────────────────────────────┘
```

---

## 🚀 Setting Up IBM Cloud MCP Server

### Step 1: Install IBM Cloud CLI

If you don't have IBM Cloud CLI installed:

**macOS:**
```bash
curl -fsSL https://clis.cloud.ibm.com/install/osx | sh
```

**Linux:**
```bash
curl -fsSL https://clis.cloud.ibm.com/install/linux | sh
```

**Windows:**
Download from [IBM Cloud CLI Downloads](https://cloud.ibm.com/docs/cli?topic=cli-install-ibmcloud-cli)

### Step 2: Verify MCP Support

Check if your IBM Cloud CLI has MCP features:

```bash
ibmcloud --help
```

Look for the **MCP OPTIONS** section:
```
MCP OPTIONS:
   --mcp-transport value    Specify transport (stdio or HTTP SSE URL)
   --mcp-tools value        Comma-delimited list of commands to expose
   --mcp-allow-write        Allow write operations (default: false)
```

### Step 3: Update IBM Cloud CLI (if needed)

If MCP options aren't available, update your CLI:

```bash
ibmcloud update
ibmcloud --help
```

### Step 4: Find IBM Cloud CLI Path

You'll need the full path for configuration:

**macOS/Linux:**
```bash
which ibmcloud
# Example output: /usr/local/bin/ibmcloud
```

**Windows:**
```bash
where ibmcloud
# Example output: C:\Program Files\IBM\Cloud\bin\ibmcloud.exe
```

---

## 🔑 Authentication Setup

### Step 1: Create IBM Cloud API Key

**Best Practice:** Use a Service ID API key (not your personal user API key)

1. Log in to [IBM Cloud Console](https://cloud.ibm.com)
2. Navigate to **Manage** → **Access (IAM)** → **Service IDs**
3. Click **Create** to create a new Service ID
4. Name it descriptively (e.g., `bob-shell-mcp-service`)
5. Click **API keys** tab → **Create**
6. Download and save the API key securely

### Step 2: Assign Permissions

Create an access group with minimal required permissions:

1. Go to **Manage** → **Access (IAM)** → **Access groups**
2. Create a new access group (e.g., `bob-shell-operators`)
3. Add your Service ID to the group
4. Assign policies based on your needs:
   - **Viewer** role for read-only operations
   - **Editor** role for resource management
   - **Operator** role for specific operations

### Step 3: Log In with API Key

```bash
ibmcloud login --apikey YOUR_API_KEY_HERE

# Set target region and resource group
ibmcloud target -r us-south -g default
```

### Step 4: Store API Key Securely

**Option 1: Environment Variable (Recommended)**
```bash
# Add to ~/.bashrc or ~/.zshrc
export IBMCLOUD_API_KEY="your-api-key-here"

# Reload shell
source ~/.bashrc  # or source ~/.zshrc
```

**Option 2: Secure Credential Manager**
Use your system's credential manager or a secrets vault.

---

## 🔧 Configuring Bob-Shell with IBM Cloud MCP

### Step 1: Locate Bob-Shell Configuration

Bob-Shell stores MCP server configurations in:
- **macOS/Linux:** `~/.config/bob-shell/mcp_servers.json`
- **Windows:** `%APPDATA%\bob-shell\mcp_servers.json`

### Step 2: Create MCP Configuration

Create or edit the configuration file:

```bash
# Create directory if it doesn't exist
mkdir -p ~/.config/bob-shell

# Create/edit configuration
nano ~/.config/bob-shell/mcp_servers.json
```

### Step 3: Add IBM Cloud MCP Server Configuration

**Basic Configuration (Read-Only):**

```json
{
  "mcpServers": {
    "ibmcloud": {
      "command": "/usr/local/bin/ibmcloud",
      "args": [
        "--mcp-transport", "stdio",
        "--mcp-tools", "resource_groups,target,is_vpcs,is_subnets,is_instances,ks_clusters"
      ],
      "env": {
        "IBMCLOUD_API_KEY": "${IBMCLOUD_API_KEY}"
      }
    }
  }
}
```

**With Write Permissions (Use with Caution):**

```json
{
  "mcpServers": {
    "ibmcloud": {
      "command": "/usr/local/bin/ibmcloud",
      "args": [
        "--mcp-transport", "stdio",
        "--mcp-allow-write",
        "--mcp-tools", "resource_groups,target,is_vpcs,is_subnets,is_instances,ks_clusters"
      ],
      "env": {
        "IBMCLOUD_API_KEY": "${IBMCLOUD_API_KEY}"
      }
    }
  }
}
```

**Configuration Explained:**
- `command`: Full path to ibmcloud CLI
- `--mcp-transport stdio`: Use standard input/output for communication
- `--mcp-tools`: Comma-separated list of IBM Cloud commands to expose
- `--mcp-allow-write`: Enable write operations (optional, use carefully)
- `env`: Environment variables (API key reference)

### Step 4: Available Tools/Commands

Common IBM Cloud commands you can expose:

**Resource Management:**
- `resource_groups` - List and manage resource groups
- `target` - Set target region and resource group

**VPC (Infrastructure as a Service):**
- `is_vpcs` - VPC management
- `is_subnets` - Subnet management
- `is_instances` - Virtual server instances
- `is_security_groups` - Security groups
- `is_floating_ips` - Floating IPs
- `is_load_balancers` - Load balancers
- `is_vpn_gateways` - VPN gateways

**Kubernetes:**
- `ks_clusters` - Kubernetes clusters
- `ks_workers` - Worker nodes
- `ks_worker_pools` - Worker pools

**Databases:**
- `cdb` - Cloud Databases

**Object Storage:**
- `cos` - Cloud Object Storage

**Code Engine:**
- `ce_apps` - Code Engine applications
- `ce_jobs` - Code Engine jobs

**Note:** Tool names follow IBM Cloud CLI command structure. For example:
- `is_vpcs` maps to `ibmcloud is vpcs`
- `ks_clusters` maps to `ibmcloud ks clusters`

### Step 5: Verify Configuration

Start Bob-Shell and verify the connection:

```bash
bob

# Inside Bob-Shell, check available MCP servers
bob> list mcp servers

# Test IBM Cloud connection
bob> list resource groups
```

Expected output:
```
✓ MCP Server: ibmcloud
✓ Status: Connected
✓ Available Tools: 6
✓ Mode: Read-only

Resource Groups:
1. default
2. production
3. staging
```

---

## 🎯 Using IBM Cloud MCP with Bob-Shell

### Example 1: List VPCs

```bash
bob> list all VPCs in my account
```

**Bob's Response:**
```
Found 3 VPCs in your IBM Cloud account:

1. production-vpc
   - ID: r006-abc123...
   - Region: us-south
   - Status: available
   - Subnets: 6

2. staging-vpc
   - ID: r006-def456...
   - Region: us-south
   - Status: available
   - Subnets: 3

3. dev-vpc
   - ID: r006-ghi789...
   - Region: us-south
   - Status: available
   - Subnets: 2
```

### Example 2: Get Resource Details

```bash
bob> show details for production-vpc
```

**Bob will:**
- Query VPC details via MCP
- Display comprehensive information
- Show related resources (subnets, security groups, etc.)

### Example 3: Kubernetes Cluster Information

```bash
bob> list kubernetes clusters and their status
```

**Bob's Response:**
```
Kubernetes Clusters:

1. production-cluster
   - ID: abc123...
   - Version: 1.28.5
   - Workers: 6
   - Status: normal
   - Region: us-south

2. staging-cluster
   - ID: def456...
   - Version: 1.28.5
   - Workers: 3
   - Status: normal
   - Region: us-south
```

### Example 4: Generate Terraform from Existing Resources

```bash
bob> generate terraform code for production-vpc
```

**Bob will:**
1. Query VPC details via MCP
2. Analyze all related resources
3. Generate complete Terraform configuration
4. Include proper dependencies and references

### Example 5: Infrastructure Analysis

```bash
bob> analyze my VPC infrastructure and suggest improvements
```

**Bob will:**
- Query all VPC resources
- Analyze configuration
- Identify security issues
- Suggest optimizations
- Provide cost estimates

---

## 🔧 Advanced Configuration

### Multiple MCP Servers

You can configure multiple MCP servers for different purposes:

```json
{
  "mcpServers": {
    "ibmcloud-vpc": {
      "command": "/usr/local/bin/ibmcloud",
      "args": [
        "--mcp-transport", "stdio",
        "--mcp-tools", "is_vpcs,is_subnets,is_instances,is_security_groups"
      ],
      "env": {
        "IBMCLOUD_API_KEY": "${IBMCLOUD_API_KEY}"
      }
    },
    "ibmcloud-kubernetes": {
      "command": "/usr/local/bin/ibmcloud",
      "args": [
        "--mcp-transport", "stdio",
        "--mcp-tools", "ks_clusters,ks_workers,ks_worker_pools"
      ],
      "env": {
        "IBMCLOUD_API_KEY": "${IBMCLOUD_API_KEY}"
      }
    },
    "ibmcloud-databases": {
      "command": "/usr/local/bin/ibmcloud",
      "args": [
        "--mcp-transport", "stdio",
        "--mcp-tools", "cdb"
      ],
      "env": {
        "IBMCLOUD_API_KEY": "${IBMCLOUD_API_KEY}"
      }
    }
  }
}
```

### Region-Specific Configuration

Configure different MCP servers for different regions:

```json
{
  "mcpServers": {
    "ibmcloud-us-south": {
      "command": "/usr/local/bin/ibmcloud",
      "args": [
        "--mcp-transport", "stdio",
        "--mcp-tools", "resource_groups,is_vpcs,ks_clusters"
      ],
      "env": {
        "IBMCLOUD_API_KEY": "${IBMCLOUD_API_KEY}",
        "IBMCLOUD_REGION": "us-south"
      }
    },
    "ibmcloud-eu-de": {
      "command": "/usr/local/bin/ibmcloud",
      "args": [
        "--mcp-transport", "stdio",
        "--mcp-tools", "resource_groups,is_vpcs,ks_clusters"
      ],
      "env": {
        "IBMCLOUD_API_KEY": "${IBMCLOUD_API_KEY}",
        "IBMCLOUD_REGION": "eu-de"
      }
    }
  }
}
```

---

## 🛡️ Security Best Practices

### 1. Use Service IDs

✅ **Do:**
- Create dedicated Service IDs for MCP access
- Use descriptive names (e.g., `bob-shell-mcp-readonly`)
- Assign minimal required permissions

❌ **Don't:**
- Use your personal user API key
- Share API keys across multiple services
- Grant excessive permissions

### 2. Read-Only by Default

✅ **Do:**
- Start with read-only configuration (no `--mcp-allow-write`)
- Only enable write operations when absolutely necessary
- Use separate configurations for read and write operations

❌ **Don't:**
- Enable write operations by default
- Use write-enabled configuration for exploration

### 3. Limit Exposed Tools

✅ **Do:**
- Only expose commands you actually need
- Keep tool list to 20-30 commands maximum
- Create specialized configurations for different tasks

❌ **Don't:**
- Expose all available commands
- Use wildcard or overly broad tool lists

### 4. Secure API Keys

✅ **Do:**
- Store API keys in environment variables
- Use system credential managers
- Rotate API keys regularly
- Monitor API key usage

❌ **Don't:**
- Hardcode API keys in configuration files
- Commit API keys to version control
- Share API keys in plain text

### 5. Monitor and Audit

```bash
# Enable audit logging
bob config set mcp.logging.audit true

# View MCP activity logs
tail -f ~/.config/bob-shell/logs/mcp-audit.log

# Review IBM Cloud activity
ibmcloud iam activity-logs
```

---

## 🐛 Troubleshooting

### Issue 1: MCP Server Not Found

**Symptom:**
```
Error: MCP server 'ibmcloud' not found
```

**Solution:**
```bash
# Verify configuration file exists
ls -la ~/.config/bob-shell/mcp_servers.json

# Check file permissions
chmod 600 ~/.config/bob-shell/mcp_servers.json

# Verify JSON syntax
cat ~/.config/bob-shell/mcp_servers.json | jq .

# Restart Bob-Shell
```

### Issue 2: Authentication Failed

**Symptom:**
```
Error: Authentication failed with IBM Cloud
```

**Solution:**
```bash
# Verify API key is set
echo $IBMCLOUD_API_KEY

# Test API key manually
ibmcloud login --apikey $IBMCLOUD_API_KEY

# Check IAM permissions
ibmcloud iam user-policies $(ibmcloud account user-preference --output json | jq -r .email)

# Regenerate API key if needed
```

### Issue 3: Command Not Available

**Symptom:**
```
Error: Tool 'is_vpcs' not available
```

**Solution:**
```bash
# Verify tool is in configuration
cat ~/.config/bob-shell/mcp_servers.json | jq '.mcpServers.ibmcloud.args'

# Check IBM Cloud CLI supports the command
ibmcloud is vpcs --help

# Update configuration to include the tool
```

### Issue 4: Slow Response Times

**Symptom:**
MCP queries take a long time to respond

**Solution:**
```bash
# Reduce number of exposed tools
# Edit mcp_servers.json and limit tools to essentials

# Check network connectivity
ping cloud.ibm.com

# Verify region is correct
ibmcloud target

# Consider using regional configuration
```

---

## 📊 Performance Optimization

### 1. Limit Tool Exposure

Only expose tools you actively use:

```json
{
  "mcpServers": {
    "ibmcloud-minimal": {
      "command": "/usr/local/bin/ibmcloud",
      "args": [
        "--mcp-transport", "stdio",
        "--mcp-tools", "is_vpcs,is_subnets,is_instances"
      ],
      "env": {
        "IBMCLOUD_API_KEY": "${IBMCLOUD_API_KEY}"
      }
    }
  }
}
```

### 2. Use Task-Specific Configurations

Create different configurations for different workflows:

```bash
# VPC management
~/.config/bob-shell/mcp_vpc.json

# Kubernetes management
~/.config/bob-shell/mcp_kubernetes.json

# Database management
~/.config/bob-shell/mcp_databases.json
```

Switch configurations as needed:
```bash
bob --mcp-config ~/.config/bob-shell/mcp_vpc.json
```

### 3. Context Window Considerations

- LLMs have limited context windows
- Limit exposed tools to 20-30 per session
- More tools = less context for actual work
- Use specialized configurations for complex tasks

---

## 🎓 Best Practices

### 1. Start Simple

Begin with basic read-only configuration:
```json
{
  "mcpServers": {
    "ibmcloud": {
      "command": "/usr/local/bin/ibmcloud",
      "args": [
        "--mcp-transport", "stdio",
        "--mcp-tools", "resource_groups,is_vpcs"
      ],
      "env": {
        "IBMCLOUD_API_KEY": "${IBMCLOUD_API_KEY}"
      }
    }
  }
}
```

### 2. Gradually Add Tools

Add tools as you need them:
```bash
# Week 1: Basic VPC
--mcp-tools resource_groups,is_vpcs

# Week 2: Add subnets
--mcp-tools resource_groups,is_vpcs,is_subnets

# Week 3: Add instances
--mcp-tools resource_groups,is_vpcs,is_subnets,is_instances
```

### 3. Document Your Configuration

Add comments to your configuration (in a separate README):

```markdown
# IBM Cloud MCP Configuration

## Tools Exposed
- `resource_groups`: List and manage resource groups
- `is_vpcs`: VPC management
- `is_subnets`: Subnet management

## Permissions Required
- Viewer role on VPC Infrastructure Services
- Viewer role on Resource Groups

## Last Updated
2026-01-06
```

### 4. Test Before Production

Always test in a development environment:

```bash
# Use dev resource group
ibmcloud target -g development

# Test MCP queries
bob> list vpcs

# Verify results
# Then switch to production
```

---

## 🎯 Next Steps

Now that you've set up IBM Cloud MCP integration:

1. **[Best Practices](./05-best-practices.md)** - Learn advanced patterns and workflows
2. **[IAC-Spec-Kit](./06-iac-spec-kit.md)** - Create detailed infrastructure specifications
3. **[Examples](../examples/)** - Try real-world integration scenarios

---

## 📚 Additional Resources

- [IBM Cloud MCP Server GitHub](https://github.com/IBM-Cloud/mcp)
- [MCP Protocol Specification](https://modelcontextprotocol.io)
- [IBM Cloud CLI Documentation](https://cloud.ibm.com/docs/cli)
- [IBM Cloud API Reference](https://cloud.ibm.com/apidocs)
- [Bob-Shell MCP Documentation](https://bob-shell.dev/docs/mcp)

---

**Ready to learn best practices?** Continue to [Best Practices Guide](./05-best-practices.md) →