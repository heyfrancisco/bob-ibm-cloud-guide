# Contributing to IBM Cloud with Bob Guide

Thank you for your interest in contributing to this guide! This document provides guidelines and instructions for contributing.

## 🎯 How to Contribute

We welcome contributions in many forms:

- 📝 **Documentation improvements**: Fix typos, clarify explanations, add examples
- 💡 **New content**: Add new guides, tutorials, or best practices
- 🐛 **Bug reports**: Report issues with examples or instructions
- ✨ **Feature requests**: Suggest new topics or improvements
- 🔧 **Code examples**: Share working Terraform configurations
- 📊 **Case studies**: Share real-world implementation stories

## 🚀 Getting Started

### 1. Fork and Clone

```bash
# Fork the repository on GitHub
# Then clone your fork
git clone https://github.com/YOUR-USERNAME/bob-ibm-cloud-guide.git
cd bob-ibm-cloud-guide

# Add upstream remote
git remote add upstream https://github.com/ORIGINAL-OWNER/bob-ibm-cloud-guide.git
```

### 2. Create a Branch

```bash
# Create a feature branch
git checkout -b feature/your-feature-name

# Or a documentation branch
git checkout -b docs/your-doc-improvement
```

### 3. Make Your Changes

Follow the guidelines below for different types of contributions.

## 📝 Documentation Guidelines

### Writing Style

- **Clear and Concise**: Use simple, direct language
- **Practical**: Include real-world examples and use cases
- **Structured**: Use headings, lists, and code blocks effectively
- **Inclusive**: Write for diverse audiences (beginners to experts)
- **Professional**: Maintain a professional but friendly tone

### Markdown Format

```markdown
# Main Heading (H1) - One per document

## Section Heading (H2)

### Subsection Heading (H3)

#### Minor Heading (H4)

**Bold text** for emphasis
*Italic text* for terms
`inline code` for commands and variables

```code blocks for examples```

> Blockquotes for important notes

- Bullet points for lists
1. Numbered lists for steps
```

### Code Examples

Always include:
- **Context**: Explain what the code does
- **Comments**: Add inline comments for complex parts
- **Complete examples**: Provide working, runnable code
- **Output**: Show expected results when relevant

```hcl
# Good example with context
# This creates a VPC with proper tagging
resource "ibm_is_vpc" "example" {
  name           = "production-vpc"
  resource_group = data.ibm_resource_group.group.id
  
  # Tags for cost tracking and management
  tags = [
    "env:production",
    "managed-by:terraform"
  ]
}
```

### Screenshots and Diagrams

- Use clear, high-resolution images
- Include alt text for accessibility
- Store images in `docs/images/` directory
- Use diagrams to explain complex architectures

## 🏗️ Adding New Content

### New Guide Structure

```markdown
# Guide Title

> **Brief description of what readers will learn**

## 📋 Overview

Brief introduction to the topic.

**What You'll Learn:**
- Key point 1
- Key point 2
- Key point 3

**Prerequisites:**
- Prerequisite 1
- Prerequisite 2

**Estimated Time:** X minutes

---

## Main Content Sections

[Your content here]

---

## 🎯 Next Steps

Links to related guides.

---

## 📚 Additional Resources

- External links
- References
```

### Adding Examples

Create a new directory under `examples/`:

```
examples/
└── your-example/
    ├── README.md          # Explanation and usage
    ├── main.tf            # Main Terraform configuration
    ├── variables.tf       # Variable definitions
    ├── outputs.tf         # Output definitions
    ├── terraform.tfvars.example  # Example values
    └── .gitignore         # Ignore sensitive files
```

## 🧪 Testing Your Changes

### Documentation Testing

1. **Spell check**: Use a spell checker
2. **Link validation**: Ensure all links work
3. **Code validation**: Test all code examples
4. **Formatting**: Preview markdown rendering

### Code Testing

```bash
# Format Terraform code
terraform fmt -recursive

# Validate configuration
terraform init
terraform validate

# Test with Bob
bob validate examples/your-example
```

## 📋 Pull Request Process

### 1. Commit Your Changes

```bash
# Stage your changes
git add .

# Commit with a descriptive message
git commit -m "docs: add guide for VPC configuration"

# Use conventional commit format:
# feat: new feature
# fix: bug fix
# docs: documentation changes
# style: formatting changes
# refactor: code refactoring
# test: adding tests
# chore: maintenance tasks
```

### 2. Push to Your Fork

```bash
git push origin feature/your-feature-name
```

### 3. Create Pull Request

1. Go to the original repository on GitHub
2. Click "New Pull Request"
3. Select your fork and branch
4. Fill in the PR template:

```markdown
## Description
Brief description of your changes

## Type of Change
- [ ] Documentation update
- [ ] New guide/tutorial
- [ ] Bug fix
- [ ] New example
- [ ] Other (please describe)

## Checklist
- [ ] I have tested the code examples
- [ ] I have checked for spelling/grammar errors
- [ ] I have followed the style guidelines
- [ ] I have added appropriate links
- [ ] My changes generate no new warnings

## Related Issues
Closes #123 (if applicable)
```

### 4. Review Process

- Maintainers will review your PR
- Address any feedback or requested changes
- Once approved, your PR will be merged

## 🎨 Style Guide

### Naming Conventions

**Files:**
- Use lowercase with hyphens: `getting-started.md`
- Be descriptive: `vpc-configuration-guide.md`

**Resources:**
- Use descriptive names: `production-vpc` not `vpc1`
- Include environment: `prod-webapp-cluster`
- Use hyphens: `my-resource-name`

**Variables:**
- Use snake_case: `vpc_name`, `subnet_count`
- Be descriptive: `database_memory_mb` not `db_mem`

### Code Style

**Terraform:**
```hcl
# Use consistent formatting
resource "ibm_is_vpc" "vpc" {
  name           = var.vpc_name
  resource_group = data.ibm_resource_group.group.id
  
  tags = var.tags
}

# Group related resources
# Add comments for complex logic
# Use variables for reusability
```

**Bash:**
```bash
#!/bin/bash
# Use shellcheck for validation
# Add error handling
set -euo pipefail

# Use descriptive variable names
readonly API_KEY="${IBMCLOUD_API_KEY}"
readonly REGION="${IC_REGION:-us-south}"
```

## 🐛 Reporting Issues

### Bug Reports

Include:
- **Description**: Clear description of the issue
- **Steps to Reproduce**: Detailed steps
- **Expected Behavior**: What should happen
- **Actual Behavior**: What actually happens
- **Environment**: OS, versions, etc.
- **Screenshots**: If applicable

### Feature Requests

Include:
- **Use Case**: Why is this needed?
- **Proposed Solution**: How should it work?
- **Alternatives**: Other approaches considered
- **Additional Context**: Any other relevant information

## 💬 Communication

- **GitHub Issues**: For bugs and feature requests
- **Pull Requests**: For code and documentation contributions
- **Discussions**: For questions and general discussion

## 📜 Code of Conduct

### Our Pledge

We are committed to providing a welcoming and inclusive environment for all contributors.

### Our Standards

**Positive behavior:**
- Using welcoming and inclusive language
- Being respectful of differing viewpoints
- Gracefully accepting constructive criticism
- Focusing on what is best for the community
- Showing empathy towards others

**Unacceptable behavior:**
- Harassment or discriminatory language
- Trolling or insulting comments
- Personal or political attacks
- Publishing others' private information
- Other unprofessional conduct

## 🏆 Recognition

Contributors will be recognized in:
- README.md contributors section
- Release notes for significant contributions
- Special mentions for outstanding contributions

## 📄 License

By contributing, you agree that your contributions will be licensed under the same license as the project (MIT License).

## 🙏 Thank You!

Your contributions help make this guide better for everyone. We appreciate your time and effort!

---

## Quick Reference

### Commit Message Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Formatting
- `refactor`: Code restructuring
- `test`: Adding tests
- `chore`: Maintenance

**Examples:**
```
docs: add VPC configuration guide
feat(examples): add Kubernetes cluster example
fix: correct subnet CIDR calculation
```

### Branch Naming

```
feature/add-vpc-guide
docs/improve-getting-started
fix/terraform-version-issue
```

### File Structure

```
bob-ibm-cloud-guide/
├── docs/
│   ├── 01-getting-started.md
│   ├── 02-terraform-basics.md
│   └── images/
├── examples/
│   ├── vpc/
│   └── kubernetes/
├── README.md
├── CONTRIBUTING.md
└── LICENSE
```

---

**Questions?** Open an issue or start a discussion. We're here to help! 🚀
