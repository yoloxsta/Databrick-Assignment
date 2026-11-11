# Azure Databricks Workspace Deployment Guide

## Complete Step-by-Step Deployment with Security Configurations

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Deployment Options](#deployment-options)
4. [Step-by-Step: Basic Deployment](#step-by-step-basic-deployment)
5. [Step-by-Step: Secure Deployment (No Public IP + VNet)](#step-by-step-secure-deployment)
6. [Post-Deployment Configuration](#post-deployment-configuration)
7. [Troubleshooting](#troubleshooting)

---

## Overview

This guide provides detailed instructions for deploying Azure Databricks workspace with various security configurations including:

- **Secure Cluster Connectivity (No Public IP)**: Enhanced security by removing public IP addresses
- **VNet Injection**: Deploy workspace in your own Virtual Network for full network control

### Deployment Wizard Tabs

```
┌────────────────────────────────────────────────────────┐
│  Create Azure Databricks Workspace                      │
├────────────────────────────────────────────────────────┤
│  1. Basics                                              │
│  2. Networking                                          │
│  3. Encryption                                          │
│  4. Security & compliance                               │
│  5. Tags                                               │
│  6. Review + create                                     │
└────────────────────────────────────────────────────────┘
```

---

## Prerequisites

### Required Before You Start

| Requirement | Details |
|------------|---------|
| **Azure Subscription** | Must have Contributor or Owner role |
| **Subscription Type** | Pay-as-you-go (not Free Trial) |
| **Resource Providers** | Microsoft.Databricks registered |
| **Quota** | Sufficient vCPU quota in your region |
| **Azure CLI** (optional) | Version 2.0+ for CLI deployment |

### Check Prerequisites

```powershell
# Login to Azure
az login

# Check subscription
az account show

# Register Databricks provider (if needed)
az provider register --namespace Microsoft.Databricks

# Check registration status
az provider show --namespace Microsoft.Databricks --query "registrationState"
```

### Required Permissions

```
Minimum Required Permissions:
──────────────────────────────────
✓ Microsoft.Databricks/workspaces/*
✓ Microsoft.Resources/subscriptions/resourceGroups/*
✓ Microsoft.Network/virtualNetworks/*
✓ Microsoft.Storage/storageAccounts/*
✓ Microsoft.Compute/*
```

---

## Deployment Options

### Option 1: Basic Deployment (Simplest)

```
┌─────────────────────────────────────────────┐
│         Basic Workspace                      │
├─────────────────────────────────────────────┤
│  ✓ Quick setup                              │
│  ✓ Managed networking                       │
│  ✓ Public endpoint accessible               │
│  ✓ No VNet configuration needed             │
│  ✓ Best for: Development, Learning          │
└─────────────────────────────────────────────┘

Configuration:
• Secure Cluster Connectivity: No
• VNet Injection: No
• Public IP: Yes
```

### Option 2: Secure Deployment (No Public IP)

```
┌─────────────────────────────────────────────┐
│    Secure Cluster Connectivity (SCC)        │
├─────────────────────────────────────────────┤
│  ✓ No public IP on clusters                 │
│  ✓ Enhanced security                        │
│  ✓ Private link support                     │
│  ✓ Managed VNet                             │
│  ✓ Best for: Production, Enterprise         │
└─────────────────────────────────────────────┘

Configuration:
• Secure Cluster Connectivity: Yes
• VNet Injection: No
• Public IP: No (removed)
```

### Option 3: VNet Injection (Full Control)

```
┌─────────────────────────────────────────────┐
│         VNet Injection                       │
├─────────────────────────────────────────────┤
│  ✓ Your own VNet                            │
│  ✓ Custom network rules                     │
│  ✓ Connect to on-premises                   │
│  ✓ Full network control                     │
│  ✓ Best for: Enterprise, Hybrid             │
└─────────────────────────────────────────────┘

Configuration:
• Secure Cluster Connectivity: Optional
• VNet Injection: Yes
• Public IP: Depends on SCC setting
```

### Option 4: Most Secure (SCC + VNet)

```
┌─────────────────────────────────────────────┐
│    SCC + VNet Injection (Most Secure)       │
├─────────────────────────────────────────────┤
│  ✓ No public IP                             │
│  ✓ Your own VNet                            │
│  ✓ Full network isolation                   │
│  ✓ Private endpoints only                   │
│  ✓ Best for: High-security environments     │
└─────────────────────────────────────────────┘

Configuration:
• Secure Cluster Connectivity: Yes
• VNet Injection: Yes
• Public IP: No
```

---

## Step-by-Step: Basic Deployment

### Tab 1: Basics

**Navigate to Azure Portal:**
1. Go to https://portal.azure.com
2. Click **Create a resource** (+)
3. Search for "Azure Databricks"
4. Select **Azure Databricks**
5. Click **Create**

**Fill in Basics:**

```
┌────────────────────────────────────────────────────┐
│                     BASICS                          │
├────────────────────────────────────────────────────┤
│                                                     │
│  Subscription *                                     │
│  ┌────────────────────────────────────────────┐   │
│  │ Select your Azure subscription              │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Resource group *                                   │
│  ┌────────────────────────────────────────────┐   │
│  │ Create new: rg-databricks-dev              │   │
│  │ OR Select existing resource group           │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Region *                                           │
│  ┌────────────────────────────────────────────┐   │
│  │ Select: (Asia Pacific) East Asia            │   │
│  │ OR: East US 2, West US 2, West Europe      │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Workspace name *                                   │
│  ┌────────────────────────────────────────────┐   │
│  │ Enter: dbw-myworkspace-dev                 │   │
│  │ (Must be globally unique)                   │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Pricing Tier *                                     │
│  ┌────────────────────────────────────────────┐   │
│  │ ○ Standard                                 │   │
│  │ ● Premium (Recommended for Unity Catalog)   │   │
│  │ ○ Trial (14-Days Free DBUs)                │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Deploy Azure Databricks workspace with Secure    │
│  Cluster Connectivity (No Public IP)?             │
│  ┌────────────────────────────────────────────┐   │
│  │ ○ Yes                                      │   │
│  │ ● No  (Select No for basic deployment)     │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│             [Next: Networking >]                    │
│                                                     │
└────────────────────────────────────────────────────┘
```

**Field Descriptions:**

| Field | Description | Example |
|-------|-------------|---------|
| **Subscription** | Azure subscription to use | My-Production-Subscription |
| **Resource group** | Container for resources | rg-databricks-prod |
| **Region** | Azure region for deployment | East US 2 |
| **Workspace name** | Unique workspace identifier | dbw-analytics-prod |
| **Pricing Tier** | Feature set | Premium (recommended) |
| **Secure Cluster Connectivity** | Security feature | No (for basic) |

**Pricing Tier Comparison:**

| Feature | Standard | Premium | Trial |
|---------|----------|---------|-------|
| Unity Catalog | No | Yes | Yes |
| Credential Passthrough | No | Yes | Yes |
| Role-Based Access Control | Limited | Full | Full |
| Audit Logs | Basic | Advanced | Advanced |
| Cost | Lower | Higher | Free 14 days |

### Tab 2: Networking (Basic)

```
┌────────────────────────────────────────────────────┐
│                   NETWORKING                        │
├────────────────────────────────────────────────────┤
│                                                     │
│  Deploy Azure Databricks workspace in your own    │
│  Virtual Network (VNet)?                          │
│  ┌────────────────────────────────────────────┐   │
│  │ ○ Yes                                      │   │
│  │ ● No  (Select No for basic deployment)     │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Note: Basic deployment uses Databricks-managed   │
│  VNet automatically.                              │
│                                                     │
│             [Next: Encryption >]                    │
│                                                     │
└────────────────────────────────────────────────────┘
```

### Tab 3: Encryption

```
┌────────────────────────────────────────────────────┐
│                   ENCRYPTION                        │
├────────────────────────────────────────────────────┤
│                                                     │
│  Enable platform-managed key encryption?           │
│  ┌────────────────────────────────────────────┐   │
│  │ ● Yes (Default - Microsoft managed keys)   │   │
│  │ ○ No (Use customer-managed keys)           │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  For basic deployment, keep default (Yes).        │
│                                                     │
│  Advanced: Customer-managed keys (CMK)            │
│  ┌────────────────────────────────────────────┐   │
│  │ Only needed for compliance requirements     │   │
│  │ Requires Azure Key Vault setup              │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│             [Next: Security & compliance >]        │
│                                                     │
└────────────────────────────────────────────────────┘
```

**Encryption Options:**

| Type | Description | Use Case |
|------|-------------|----------|
| **Platform-managed** | Microsoft manages keys | Default, most common |
| **Customer-managed** | You manage keys in Key Vault | Compliance, regulated industries |

### Tab 4: Security & Compliance

```
┌────────────────────────────────────────────────────┐
│              SECURITY & COMPLIANCE                  │
├────────────────────────────────────────────────────┤
│                                                     │
│  System-assigned identity                          │
│  ┌────────────────────────────────────────────┐   │
│  │ Status: (on) - Read-only                    │   │
│  │ Automatically enabled for managed identity  │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  User-assigned identity                            │
│  ┌────────────────────────────────────────────┐   │
│  │ Select user-assigned identity (optional)    │   │
│  │ ○ None selected                             │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Storage firewall                                  │
│  ┌────────────────────────────────────────────┐   │
│  │ ○ Enable storage firewall                   │   │
│  │ ● Disable (Default)                         │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Compliance standards                              │
│  ┌────────────────────────────────────────────┐   │
│  │ No additional compliance settings needed    │   │
│  │ for basic deployment                        │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│             [Next: Tags >]                          │
│                                                     │
└────────────────────────────────────────────────────┘
```

### Tab 5: Tags

```
┌────────────────────────────────────────────────────┐
│                      TAGS                           │
├────────────────────────────────────────────────────┤
│                                                     │
│  Add tags to organize your resources:              │
│                                                     │
│  ┌──────────────┬──────────────────────────────┐  │
│  │ Name         │ Value                         │  │
│  ├──────────────┼──────────────────────────────┤  │
│  │ Environment  │ Development                   │  │
│  ├──────────────┼──────────────────────────────┤  │
│  │ Department   │ Data Engineering              │  │
│  ├──────────────┼──────────────────────────────┤  │
│  │ CostCenter   │ 12345                         │  │
│  ├──────────────┼──────────────────────────────┤  │
│  │ Owner        │ data-team@company.com         │  │
│  └──────────────┴──────────────────────────────┘  │
│                                                     │
│  [ + Add tag ]                                      │
│                                                     │
│             [Next: Review + create >]               │
│                                                     │
└────────────────────────────────────────────────────┘
```

**Recommended Tags:**

| Tag Name | Example Value | Purpose |
|----------|---------------|---------|
| Environment | Production / Dev / Test | Resource classification |
| Department | Data-Engineering | Cost allocation |
| CostCenter | CC-12345 | Billing tracking |
| Owner | team@company.com | Contact information |
| Application | Analytics-Platform | Application identification |

### Tab 6: Review + Create

```
┌────────────────────────────────────────────────────┐
│                 REVIEW + CREATE                     │
├────────────────────────────────────────────────────┤
│                                                     │
│  Validation                                         │
│  ┌────────────────────────────────────────────┐   │
│  │ ✓ Validation passed                         │   │
│  │ ✓ All required fields completed             │   │
│  │ ✓ Quota available                           │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Summary                                            │
│  ┌────────────────────────────────────────────┐   │
│  │ Subscription:    My-Subscription             │   │
│  │ Resource group:  rg-databricks-dev          │   │
│  │ Region:          East US 2                   │   │
│  │ Workspace name:  dbw-myworkspace-dev        │   │
│  │ Pricing Tier:    Premium                     │   │
│  │ Secure Cluster:  No                          │   │
│  │ VNet Injection:  No                          │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Estimated cost: $0.40/hour (approximate)          │
│                                                     │
│  Automation                                         │
│  ┌────────────────────────────────────────────┐   │
│  │ [Download a template for automation]        │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│                [ Create ]                           │
│                                                     │
└────────────────────────────────────────────────────┘
```

**Deployment Process:**

```
Deployment Progress:
──────────────────────────────────
⏳ Validating deployment...
⏳ Creating resource group...
⏳ Creating storage account...
⏳ Creating Databricks workspace...
⏳ Configuring workspace...
✓ Deployment complete!

Estimated time: 3-5 minutes
```

**Post-Deployment:**

```
┌────────────────────────────────────────────────────┐
│          DEPLOYMENT COMPLETE                        │
├────────────────────────────────────────────────────┤
│                                                     │
│  Your deployment is complete                       │
│                                                     │
│  Resource group: rg-databricks-dev                 │
│                                                     │
│  Resources created:                                │
│  ├── Azure Databricks workspace                    │
│  ├── Storage account (managed)                     │
│  └── Network resources (managed)                   │
│                                                     │
│  [Go to resource]                                   │
│                                                     │
└────────────────────────────────────────────────────┘
```

---

## Step-by-Step: Secure Deployment (No Public IP + VNet)

### Overview

This section covers the most secure deployment with:
- **Secure Cluster Connectivity (No Public IP)**: Yes
- **VNet Injection**: Yes

### Prerequisites for Secure Deployment

**Required Before Starting:**

```powershell
# 1. Create Resource Group
az group create \
  --name rg-databricks-secure \
  --location eastus2

# 2. Create Virtual Network with two subnets
az network vnet create \
  --resource-group rg-databricks-secure \
  --name vnet-databricks \
  --address-prefix 10.0.0.0/16 \
  --subnet-name public-subnet \
  --subnet-prefix 10.0.0.0/18

# 3. Add private subnet
az network vnet subnet create \
  --resource-group rg-databricks-secure \
  --vnet-name vnet-databricks \
  --name private-subnet \
  --address-prefix 10.0.64.0/18

# 4. Disable private subnet features
az network vnet subnet update \
  --resource-group rg-databricks-secure \
  --vnet-name vnet-databricks \
  --name private-subnet \
  --disable-private-link-service-network-policies true

# 5. Create Network Security Group
az network nsg create \
  --resource-group rg-databricks-secure \
  --name nsg-databricks
```

### Tab 1: Basics (Secure Deployment)

```
┌────────────────────────────────────────────────────┐
│                     BASICS                          │
├────────────────────────────────────────────────────┤
│                                                     │
│  Subscription *                                     │
│  ┌────────────────────────────────────────────┐   │
│  │ Select your Azure subscription              │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Resource group *                                   │
│  ┌────────────────────────────────────────────┐   │
│  │ Select: rg-databricks-secure                │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Region *                                           │
│  ┌────────────────────────────────────────────┐   │
│  │ Select: East US 2                           │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Workspace name *                                   │
│  ┌────────────────────────────────────────────┐   │
│  │ Enter: dbw-secure-prod                      │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Pricing Tier *                                     │
│  ┌────────────────────────────────────────────┐   │
│  │ ● Premium (Required for SCC)                │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Deploy Azure Databricks workspace with Secure    │
│  Cluster Connectivity (No Public IP)?             │
│  ┌────────────────────────────────────────────┐   │
│  │ ● Yes  ← IMPORTANT: Select Yes              │   │
│  │ ○ No                                       │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│             [Next: Networking >]                    │
│                                                     │
└────────────────────────────────────────────────────┘
```

**Important Notes:**

⚠️ **Secure Cluster Connectivity requires Premium tier**

✓ When you select "Yes" for SCC:
- No public IP addresses on clusters
- Enhanced security posture
- Requires Private Link for access

### Tab 2: Networking (Secure Deployment with VNet)

```
┌────────────────────────────────────────────────────┐
│                   NETWORKING                        │
├────────────────────────────────────────────────────┤
│                                                     │
│  Deploy Azure Databricks workspace in your own    │
│  Virtual Network (VNet)?                          │
│  ┌────────────────────────────────────────────┐   │
│  │ ● Yes  ← IMPORTANT: Select Yes              │   │
│  │ ○ No                                       │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  ┌────────────────────────────────────────────┐   │
│  │ When you select Yes, additional fields      │   │
│  │ will appear for VNet configuration          │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Virtual Network *                                  │
│  ┌────────────────────────────────────────────┐   │
│  │ Select: vnet-databricks                     │   │
│  │ (Created in prerequisites)                  │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Public Subnet Name *                               │
│  ┌────────────────────────────────────────────┐   │
│  │ Select: public-subnet                       │   │
│  │ Purpose: Hosts the Databricks control plane │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Public Subnet CIDR                                │
│  ┌────────────────────────────────────────────┐   │
│  │ Range: 10.0.0.0/18                          │   │
│  │ (Auto-populated from subnet)                │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Private Subnet Name *                              │
│  ┌────────────────────────────────────────────┐   │
│  │ Select: private-subnet                      │   │
│  │ Purpose: Hosts worker nodes                 │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Private Subnet CIDR                               │
│  ┌────────────────────────────────────────────┐   │
│  │ Range: 10.0.64.0/18                         │   │
│  │ (Auto-populated from subnet)                │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│             [Next: Encryption >]                    │
│                                                     │
└────────────────────────────────────────────────────┘
```

**VNet Architecture:**

```
Your Azure Subscription
│
└── vnet-databricks (10.0.0.0/16)
    │
    ├── public-subnet (10.0.0.0/18)
    │   ├── Purpose: Control plane communication
    │   ├── Hosts: Databricks control plane components
    │   └── Outbound: Internet access for control plane
    │
    └── private-subnet (10.0.64.0/18)
        ├── Purpose: Compute plane (clusters)
        ├── Hosts: Worker nodes, driver nodes
        ├── No public IP (SCC enabled)
        └── Private Link for connectivity
```

**Subnet Requirements:**

| Requirement | Public Subnet | Private Subnet |
|-------------|---------------|----------------|
| **Size** | Minimum /26 (64 IPs) | Minimum /26 (64 IPs) |
| **Delegated** | Not required | Not required |
| **Private Link Policies** | Default | Must be disabled |
| **NSG** | Optional | Optional |

### Tab 3: Encryption (Secure Deployment)

```
┌────────────────────────────────────────────────────┐
│                   ENCRYPTION                        │
├────────────────────────────────────────────────────┤
│                                                     │
│  For maximum security with customer-managed keys:  │
│                                                     │
│  Enable platform-managed key encryption?           │
│  ┌────────────────────────────────────────────┐   │
│  │ ○ Yes (Default)                            │   │
│  │ ● No (Enable customer-managed keys)        │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  If you selected No, configure:                    │
│                                                     │
│  Key vault *                                        │
│  ┌────────────────────────────────────────────┐   │
│  │ Select your Azure Key Vault                 │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Key *                                              │
│  ┌────────────────────────────────────────────┐   │
│  │ Select the encryption key                   │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Version                                            │
│  ┌────────────────────────────────────────────┐   │
│  │ Select specific version or auto-update      │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Note: Customer-managed keys require:              │
│  • Azure Key Vault with purge protection           │
│  • RSA key with 2048-bit minimum                   │
│  • Key Vault in same region                        │
│                                                     │
│             [Next: Security & compliance >]        │
│                                                     │
└────────────────────────────────────────────────────┘
```

**Customer-Managed Keys Setup (Optional):**

```powershell
# Create Key Vault with purge protection
az keyvault create \
  --name kv-databricks-encryption \
  --resource-group rg-databricks-secure \
  --location eastus2 \
  --enable-purge-protection true \
  --enable-soft-delete true

# Create encryption key
az keyvault key create \
  --name databricks-encryption-key \
  --vault-name kv-databricks-encryption \
  --kty RSA \
  --size 2048

# Grant Databricks access to key
# (Workspace managed identity will be created during deployment)
```

### Tab 4: Security & Compliance (Secure Deployment)

```
┌────────────────────────────────────────────────────┐
│              SECURITY & COMPLIANCE                  │
├────────────────────────────────────────────────────┤
│                                                     │
│  System-assigned identity                          │
│  ┌────────────────────────────────────────────┐   │
│  │ Status: (on) - Automatically enabled        │   │
│  │ Required for VNet injection                 │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  User-assigned identity                            │
│  ┌────────────────────────────────────────────┐   │
│  │ Select managed identity (optional)          │   │
│  │ For enhanced security scenarios             │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Storage firewall                                  │
│  ┌────────────────────────────────────────────┐   │
│  │ ● Enable storage firewall (Recommended)     │   │
│  │ ○ Disable                                   │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Private endpoint settings                         │
│  ┌────────────────────────────────────────────┐   │
│  │ Configure after deployment:                 │   │
│  │ • Workspace private endpoint                │   │
│  │ • Storage private endpoint                  │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│             [Next: Tags >]                          │
│                                                     │
└────────────────────────────────────────────────────┘
```

### Tab 5: Tags (Secure Deployment)

```
┌────────────────────────────────────────────────────┐
│                      TAGS                           │
├────────────────────────────────────────────────────┤
│                                                     │
│  Add tags for resource management:                 │
│                                                     │
│  ┌──────────────┬──────────────────────────────┐  │
│  │ Name         │ Value                         │  │
│  ├──────────────┼──────────────────────────────┤  │
│  │ Environment  │ Production                    │  │
│  ├──────────────┼──────────────────────────────┤  │
│  │ Security     │ High                          │  │
│  ├──────────────┼──────────────────────────────┤  │
│  │ NetworkType  │ VNet-Injected-SCC             │  │
│  ├──────────────┼──────────────────────────────┤  │
│  │ Compliance   │ Required                      │  │
│  └──────────────┴──────────────────────────────┘  │
│                                                     │
│             [Next: Review + create >]               │
│                                                     │
└────────────────────────────────────────────────────┘
```

### Tab 6: Review + Create (Secure Deployment)

```
┌────────────────────────────────────────────────────┐
│                 REVIEW + CREATE                     │
├────────────────────────────────────────────────────┤
│                                                     │
│  Validation                                         │
│  ┌────────────────────────────────────────────┐   │
│  │ ✓ Validation passed                         │   │
│  │ ✓ VNet configuration valid                  │   │
│  │ ✓ Subnet delegation verified                │   │
│  │ ✓ Quota available in region                 │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Summary                                            │
│  ┌────────────────────────────────────────────┐   │
│  │ Subscription:    My-Subscription             │   │
│  │ Resource group:  rg-databricks-secure       │   │
│  │ Region:          East US 2                   │   │
│  │ Workspace name:  dbw-secure-prod            │   │
│  │ Pricing Tier:    Premium                     │   │
│  │ Secure Cluster:  Yes                         │   │
│  │ VNet Injection:  Yes                         │   │
│  │ VNet:            vnet-databricks             │   │
│  │ Public Subnet:   public-subnet               │   │
│  │ Private Subnet:  private-subnet              │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Security Features Enabled:                        │
│  ┌────────────────────────────────────────────┐   │
│  │ ✓ No public IP addresses                    │   │
│  │ ✓ Custom VNet                               │   │
│  │ ✓ Network isolation                         │   │
│  │ ✓ Storage firewall                          │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
│  Estimated cost: $0.50/hour (approximate)          │
│                                                     │
│                [ Create ]                           │
│                                                     │
└────────────────────────────────────────────────────┘
```

**Deployment Time:**

```
Secure Deployment Timeline:
──────────────────────────────────
⏳ 0-2 min: Validating configuration
⏳ 2-5 min: Creating workspace resources
⏳ 5-10 min: Configuring networking
⏳ 10-15 min: Enabling secure cluster connectivity
⏳ 15-20 min: Finalizing deployment
✓ 20-25 min: Deployment complete

Note: Secure deployments take longer than basic
```

---

## Post-Deployment Configuration

### 1. Access Your Workspace

**For Basic Deployment:**
```
1. Navigate to your Databricks resource in Azure Portal
2. Click "Launch Workspace"
3. Workspace opens in new tab
4. You're automatically logged in with Azure AD
```

**For Secure Deployment:**

```
⚠️ Direct access not available with SCC + VNet

Required: Set up Private Endpoint first

Steps:
1. Create Private Endpoint for workspace
2. Configure DNS resolution
3. Access via private network only
```

### 2. Configure Private Link (Secure Deployment)

```powershell
# Create Private Endpoint for workspace
az network private-endpoint create \
  --name pe-databricks-workspace \
  --resource-group rg-databricks-secure \
  --vnet-name vnet-databricks \
  --subnet private-subnet \
  --private-connection-resource-id /subscriptions/<sub-id>/resourceGroups/rg-databricks-secure/providers/Microsoft.Databricks/workspaces/dbw-secure-prod \
  --group-id databricks_ui_api \
  --connection-name databricks-connection

# Configure Private DNS Zone
az network private-dns zone create \
  --resource-group rg-databricks-secure \
  --name privatelink.azuredatabricks.net

# Link DNS zone to VNet
az network private-dns link vnet create \
  --resource-group rg-databricks-secure \
  --zone-name privatelink.azuredatabricks.net \
  --name dns-link \
  --virtual-network vnet-databricks \
  --registration-enabled false
```

### 3. Enable Unity Catalog

```
1. In Databricks workspace, click "Catalog" in sidebar
2. Click "Create Metastore"
3. Select region (must match workspace region)
4. Provide metastore name
5. Click "Create"
6. Assign metastore to workspace
```

### 4. Create First Cluster

**For Basic Workspace:**
```
1. Click "Compute" in sidebar
2. Click "Create Compute"
3. Configure:
   - Name: dev-cluster
   - Runtime: Latest LTS
   - Node type: Standard_DS3_v2
   - Workers: 2
   - Auto-termination: 20 minutes
4. Click "Create Compute"
```

**For Secure Workspace:**
```
Clusters will automatically:
✓ Use VNet-injected subnets
✓ Have no public IP (SCC)
✓ Be isolated in your network
```

### 5. Configure Network Security Groups (Optional)

```powershell
# Create NSG rules for restricted access
az network nsg rule create \
  --resource-group rg-databricks-secure \
  --nsg-name nsg-databricks \
  --name Allow-Databricks-Control-Plane \
  --priority 100 \
  --direction Inbound \
  --source-address-prefixes 'AzureDatabricks' \
  --destination-port-ranges 443 \
  --protocol Tcp \
  --access Allow

# Allow communication between subnets
az network nsg rule create \
  --resource-group rg-databricks-secure \
  --nsg-name nsg-databricks \
  --name Allow-Internal-Communication \
  --priority 110 \
  --direction Inbound \
  --source-address-prefixes '10.0.0.0/16' \
  --destination-port-ranges '*' \
  --protocol '*' \
  --access Allow
```

---

## Troubleshooting

### Common Deployment Issues

**Issue 1: "Quota exceeded" error**

```
Solution:
1. Check your vCPU quota
   az vm list-usage --location eastus2 -o table

2. Request quota increase
   - Go to Azure Portal → Subscriptions
   - Select your subscription
   - Click "Usage + quotas"
   - Find required VM series
   - Click "Request quota increase"

3. Or select a different region with available quota
```

**Issue 2: "Subnet delegation failed"**

```
Solution:
1. Ensure subnets don't have conflicting delegations
2. Disable private link network policies on private subnet:
   az network vnet subnet update \
     --name private-subnet \
     --vnet-name vnet-databricks \
     --resource-group rg-databricks-secure \
     --disable-private-link-service-network-policies true
```

**Issue 3: "Workspace deployment failed"**

```
Check:
1. Resource provider registration
   az provider show --namespace Microsoft.Databricks

2. If not registered:
   az provider register --namespace Microsoft.Databricks

3. Wait 10-15 minutes and retry deployment
```

**Issue 4: "Cannot access workspace after deployment"**

```
For Basic Deployment:
- Check if public access is enabled
- Verify Azure AD permissions
- Try different browser/incognito mode

For Secure Deployment:
- Verify Private Endpoint is created
- Check DNS resolution
- Ensure you're on the correct network
```

**Issue 5: "VNet peering conflicts"**

```
Solution:
1. Ensure no overlapping IP ranges
2. Verify VNet peering is properly configured
3. Check route tables don't conflict
```

### Validation Commands

```powershell
# Check workspace status
az databricks workspace show \
  --resource-group rg-databricks-secure \
  --name dbw-secure-prod \
  --query provisioningState

# List workspace details
az databricks workspace list \
  --resource-group rg-databricks-secure

# Check VNet configuration
az network vnet show \
  --resource-group rg-databricks-secure \
  --name vnet-databricks \
  --query subnets

# Verify Private Endpoint
az network private-endpoint list \
  --resource-group rg-databricks-secure
```

---

## Deployment Checklist

### Basic Deployment

```
☐ Prerequisites
  ☐ Azure subscription (Pay-as-you-go)
  ☐ Resource provider registered
  ☐ Sufficient quota

☐ Deployment
  ☐ Resource group created
  ☐ Region selected
  ☐ Workspace name unique
  ☐ Pricing tier selected
  ☐ Tags added

☐ Post-Deployment
  ☐ Workspace accessible
  ☐ Unity Catalog enabled
  ☐ First cluster created
  ☐ Notebook created
```

### Secure Deployment

```
☐ Prerequisites
  ☐ Azure subscription (Pay-as-you-go)
  ☐ Resource provider registered
  ☐ VNet created with correct subnets
  ☐ NSG configured
  ☐ Key Vault created (if using CMK)

☐ Deployment
  ☐ SCC enabled (Yes)
  ☐ VNet injection enabled (Yes)
  ☐ Subnets selected correctly
  ☐ CMK configured (optional)
  ☐ Storage firewall enabled

☐ Post-Deployment
  ☐ Private Endpoint created
  ☐ DNS configured
  ☐ Network access verified
  ☐ Unity Catalog enabled
  ☐ Secure cluster created
```

---

## Summary

### Decision Matrix

| Requirement | Basic | SCC Only | VNet Only | SCC + VNet |
|-------------|-------|----------|-----------|------------|
| **Setup Complexity** | Low | Medium | Medium | High |
| **Security Level** | Basic | High | High | Highest |
| **Network Control** | None | Limited | Full | Full |
| **Public IP** | Yes | No | Yes | No |
| **Private Access** | Optional | Required | Optional | Required |
| **Use Case** | Dev/Learning | Production | Enterprise | Regulated |
| **Cost** | Lowest | Medium | Medium | Highest |

### Next Steps

1. **After Basic Deployment:**
   - Create notebooks
   - Load sample data
   - Build first pipeline
   - Try MLflow

2. **After Secure Deployment:**
   - Configure Private Link
   - Set up VPN/ExpressRoute (if hybrid)
   - Implement network security
   - Create secure clusters
   - Deploy production workloads

---

**Document Version**: 1.0  
**Last Updated**: November 2026

**Related Documentation:**
- Azure Databricks Learning Guide
- Demo Project Guide
- Theory Guide (Burmese)
