# System Initiative Windows + Azure Setup Documentation Package

**Version:** 2.0
**Created:** December 2025
**Organization:** American Sound
**Author:** Doug Schaefer (dougschaefer@asei.com)

---

## Package Overview

This documentation package covers the complete setup of System Initiative on Windows 11 with Azure as the cloud provider, including the WSL2 and Docker plumbing that most SI documentation assumes you already have working, the Azure service principal configuration that differs from the AWS-focused official docs, and two methods for integrating Claude as an AI agent (the official CLI method and a community-discovered Desktop method). The package was built from real-world implementation at American Sound, so it documents the actual failure modes and configuration mistakes we encountered rather than just the happy path.

The package includes a full reference guide, a condensed checklist for tracking progress through the setup, and two PowerShell scripts that automate the Azure service principal creation and environment validation steps.

---

## What's Included

### Documentation Files

The main reference guide covers the complete setup end-to-end, while the checklist distills the same process into a trackable format for teams running through it across multiple machines.

**[SI-Setup-Guide-Windows-Azure-v2.md](SI-Setup-Guide-Windows-Azure-v2.md)** (Main Reference)
- Complete 50+ page setup guide with both Claude Code CLI and Claude Desktop methods
- Troubleshooting section based on actual deployment issues
- Reference information and best practices

**[SI-Quick-Start-Checklist-v2.md](SI-Quick-Start-Checklist-v2.md)** (Quick Reference)
- Condensed checklist format with checkboxes for progress tracking
- Essential commands and quick troubleshooting reference

### PowerShell Scripts

The scripts automate the two most error-prone manual steps in the setup process.

**[setup-azure-service-principal-v2.ps1](setup-azure-service-principal-v2.ps1)** automates the Azure service principal creation, RBAC permission assignment, and credential export so you avoid the Client ID vs Object ID and Secret Value vs Secret ID confusion that causes most first-time authentication failures.

**[validate-environment-v2.ps1](validate-environment-v2.ps1)** checks WSL2, Docker, Azure CLI, Node.js, SI CLI installation, and optional Claude integration, providing actionable feedback for any component that is missing or misconfigured.

---

## Quick Start

### New to System Initiative?

1. **Start here:** Read [SI-Setup-Guide-Windows-Azure-v2.md](SI-Setup-Guide-Windows-Azure-v2.md)
2. **Use checklist:** Follow [SI-Quick-Start-Checklist-v2.md](SI-Quick-Start-Checklist-v2.md)
3. **Run scripts:** Use PowerShell scripts to automate setup

### Troubleshooting Existing Setup?

1. **Run validation:** Execute `validate-environment-v2.ps1` in PowerShell
2. **Check results:** Review any failures or warnings
3. **Reference guide:** See troubleshooting section in main guide

---

## Setup Process Overview

The full setup runs through four phases, each building on the last.

### Phase 1: Base System (30-45 minutes)

Install WSL2 with Ubuntu, Docker Desktop with WSL2 integration, Azure CLI, and the SI CLI inside WSL2. Most issues at this stage come from WSL2 not being properly enabled or Docker Desktop not having WSL2 integration turned on.

### Phase 2: Azure Configuration (15-20 minutes)

Create an Azure service principal with the correct RBAC permissions, generate credentials, and validate access. The automated script handles this, but if you are doing it manually, the critical detail is that you need the Application (client) ID (not the Object ID) and the secret Value (not the Secret ID).

### Phase 3: Claude Integration (10-30 minutes)

Choose one of two methods. Claude Code CLI is the official, supported method and takes 10-15 minutes. Claude Desktop is an unofficial community-discovered method that takes 20-30 minutes and may break with updates, but provides a visual chat interface.

### Phase 4: Connect and Test (10-15 minutes)

Create an SI account and workspace, configure Azure credentials, create test infrastructure, and validate the end-to-end workflow.

**Total Time:** 1-2 hours for complete setup

---

## Critical Concepts

### Token Confusion

System Initiative uses confusing token terminology that trips up most new users. API Tokens are JWT format (long strings) used for programmatic access and AI agents. Workspace Tokens are ULID format (short strings) used for the web interface. For AI integration, you need the API token, stored in the `SI_API_TOKEN` environment variable.

### Service Principal Permissions

The service principal needs Azure RBAC roles (Contributor or custom) on the subscriptions or resources it manages. It does not need Entra ID administrative roles, Microsoft Graph API permissions, or Global Administrator access. The App Registration in Entra ID creates the identity, the Client Secret provides the password, and the RBAC role assignment on the subscription provides the actual authorization.

### Azure Credential Values

The most common authentication failure comes from using the wrong values when configuring SI's Microsoft Credential.

| SI Field | Azure Portal Location | Common Mistake |
|----------|----------------------|----------------|
| **Client ID** | Application (client) ID | Do not use Object ID |
| **Client Secret** | Secret VALUE | Do not use Secret ID |
| **Tenant ID** | Directory (tenant) ID | Straightforward |

The Client Secret is only shown once when created. Copy the Value column (the 30+ character string), not the Secret ID (which is a GUID).

### WSL2 vs Windows

WSL2 and Windows are two separate environments that share a filesystem but not a tool installation. Install Azure CLI on Windows (PowerShell), Docker is managed by Docker Desktop (runs in WSL2), and Node.js, SI CLI, and Claude Code all install inside WSL2.

### Claude Code vs Claude Desktop

| Aspect | Claude Code CLI | Claude Desktop GUI |
|--------|----------------|-------------------|
| **Official Support** | Yes | No |
| **Setup Complexity** | Simple | Complex |
| **Interface** | Terminal | Chat GUI |
| **Best For** | Automation, scripting | Exploration, learning |
| **Updates** | Less likely to break | May break with updates |

Start with Claude Code CLI if you are comfortable with terminals. Use Claude Desktop if you need the visual interface and accept the maintenance burden of an unofficial integration method.

---

## Prerequisites

### Required Software

You need Windows 11 Pro or Enterprise with WSL2 and Ubuntu, Docker Desktop for Windows in WSL2 mode, Azure CLI for Windows, Node.js in WSL2, and the System Initiative CLI in WSL2.

### Required Accounts

An Azure subscription with Owner or Contributor access, a System Initiative account (free tier available), and an Anthropic Claude account for AI integration.

### Hardware Requirements

16GB RAM minimum (32GB recommended), 50GB free disk space, and a CPU with virtualization support enabled in BIOS.

---

## Validation and Testing

### Before You Begin

Run the validation script to check prerequisites before starting the setup process.

```powershell
.\validate-environment-v2.ps1
```

### After Setup

Verify end-to-end functionality by confirming WSL2 and Docker are working, Azure CLI is authenticated, SI CLI is functional, Claude can query the SI workspace, and you can create Azure resources from SI that appear in the Azure Portal.
