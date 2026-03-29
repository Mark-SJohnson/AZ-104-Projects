# Azure RBAC & Governance Structure
### AZ-104 Hands-On Project | Identity & Governance Domain

![Azure](https://img.shields.io/badge/Azure-Entra_ID-0078D4?style=flat&logo=microsoftazure&logoColor=white)
![RBAC](https://img.shields.io/badge/RBAC-Custom_Roles-0078D4?style=flat&logo=microsoftazure&logoColor=white)
![Policy](https://img.shields.io/badge/Azure-Policy-0078D4?style=flat&logo=microsoftazure&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner--Intermediate-yellow?style=flat)
![Time](https://img.shields.io/badge/Est._Time-1--2_hours-green?style=flat)
![Cost](https://img.shields.io/badge/Cost-Free-brightgreen?style=flat)

---

## Overview

This project simulates a real-world company access structure in Microsoft Azure using Entra ID groups, scoped RBAC role assignments, a custom role, and Azure Policy governance controls. It mirrors identity and access management workflows common in regulated industries such as healthcare.

**Domains covered:** Identity & Governance (AZ-104 — 20–25% of exam weight)

---

## What I Built

- Three department groups in Entra ID simulating Finance, Engineering, and HR teams
- Scoped RBAC role assignments giving each department access only to their own resource group
- A custom RBAC role (`VM Restart Operator`) with least-privilege permissions scoped to virtual machine actions only
- Two Azure Policy assignments enforcing tagging standards and location restrictions
- End-to-end RBAC validation using test user accounts in isolated browser sessions

---

## Architecture

```
Subscription
│
├── rg-finance-resources       ← grp-finance (Reader)
├── rg-engineering-resources   ← grp-engineering (Contributor + VM Restart Operator)
├── rg-hr-resources            ← grp-hr (Reader)
└── rg-iam-demo                ← Project container
```

**Azure Policies applied at subscription scope:**
- Require `Environment` tag on all resources (Audit mode)
- Allowed locations: East US and East US 2 only (Audit mode)

---

## Resources Created

| Resource | Type | Purpose |
|---|---|---|
| grp-finance | Entra ID Security Group | Finance department |
| grp-engineering | Entra ID Security Group | Engineering department |
| grp-hr | Entra ID Security Group | HR department |
| rg-finance-resources | Resource Group | Finance scoped environment |
| rg-engineering-resources | Resource Group | Engineering scoped environment |
| rg-hr-resources | Resource Group | HR scoped environment |
| VM Restart Operator | Custom RBAC Role | Least-privilege VM start/restart only |
| Require Environment Tag | Azure Policy | Enforces tagging compliance |
| Allowed Locations - US Only | Azure Policy | Restricts deployments to East US regions |

---

## RBAC Role Assignments

| Group | Role | Scope | Can Create Resources | Can Delete Resources | Can Manage Access |
|---|---|---|---|---|---|
| grp-finance | Reader | rg-finance-resources | ❌ | ❌ | ❌ |
| grp-engineering | Contributor | rg-engineering-resources | ✅ | ✅ | ❌ |
| grp-engineering | VM Restart Operator | Subscription | ❌ | ❌ | ❌ |
| grp-hr | Reader | rg-hr-resources | ❌ | ❌ | ❌ |

---

## Custom Role: VM Restart Operator

```json
{
  "Name": "VM Restart Operator",
  "Description": "Allows starting and restarting virtual machines only",
  "Actions": [
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/<subscription-id>"
  ]
}
```

**Why this matters:** Rather than assigning Contributor (which grants full resource management), this custom role grants only the two specific actions an operator needs. This is least-privilege access in practice — a core principle in both AZ-104 and HIPAA security controls.

---

## Azure Policies

### Policy 1 — Require Environment Tag
- **Definition:** Require a tag on resources
- **Parameter:** Tag name = `Environment`
- **Scope:** Subscription
- **Effect:** Audit (non-blocking)
- **Purpose:** Flags any resource missing the Environment tag. In production this would be set to Deny to enforce compliance before deployment.

### Policy 2 — Allowed Locations
- **Definition:** Allowed locations
- **Parameter:** East US, East US 2
- **Scope:** Subscription
- **Effect:** Audit (non-blocking)
- **Purpose:** Restricts deployments to US regions only. Maps directly to HIPAA data residency requirements — PHI must remain within defined geographic boundaries.

---

## Steps Completed

- [x] Created resource group `rg-iam-demo`
- [x] Created department groups in Entra ID (grp-finance, grp-engineering, grp-hr)
- [x] Created test users and assigned to groups
- [x] Created department-scoped resource groups
- [x] Assigned built-in RBAC roles scoped to each resource group
- [x] Created custom RBAC role `VM Restart Operator`
- [x] Assigned custom role to grp-engineering
- [x] Created Azure Policy for tag enforcement
- [x] Created Azure Policy for allowed locations
- [x] Validated RBAC with test user login (private browser)
- [x] Reviewed Policy Compliance dashboard
- [x] Documented structure and screenshots

---

## Validation Results

**RBAC Test — Access Granted**
Logged in as test user assigned to grp-finance. Successfully accessed `rg-finance-resources`. Resource group opened as expected with read-only permissions confirmed (no Create button visible).

**RBAC Test — Access Denied**
Attempted to access `rg-engineering-resources` as the same grp-finance test user. Received authorization error — access correctly denied.

**Policy Compliance**
Resources without the `Environment` tag flagged as non-compliant in the Policy Compliance dashboard. Allowed Locations policy confirmed active at subscription scope.

> 📸 Screenshots of all validation results available in the `/screenshots` folder

---

## Key Concepts Demonstrated

**RBAC is additive** — a user receives the union of all role permissions assigned to them directly or through group membership. There is no explicit deny in standard RBAC (unless using Azure Policy deny assignments).

**Role assignment scope matters** — assigning a role at the subscription level grants access to everything underneath it. Assigning at the resource group level scopes it to only that container. Always assign at the lowest scope required.

**Custom roles fill the gap between built-ins** — Contributor is too broad for a VM operator who only needs to restart machines. Custom roles allow you to grant exactly the permissions needed and nothing more.

**Azure Policy evaluates, not enforces by default** — Audit mode flags non-compliant resources without blocking deployments. In regulated environments like healthcare, policies are set to Deny after a compliance window to enforce controls going forward.

---

## Healthcare / Compliance Angle

This project maps directly to real-world HIPAA security controls:

- **RBAC scoping** mirrors the HIPAA minimum necessary standard — users should only access the PHI and systems they need for their role
- **Allowed Locations policy** enforces data residency — PHI must remain within defined geographic boundaries under HIPAA
- **Tag enforcement policy** supports audit logging requirements — tagged resources are easier to track in compliance reports
- **Custom roles** demonstrate the ability to implement least-privilege access at a granular level, a requirement under HIPAA's access control standard (§164.312(a)(1))

---

## Technologies Used

- Microsoft Azure Portal
- Microsoft Entra ID (formerly Azure Active Directory)
- Azure Role-Based Access Control (RBAC)
- Azure Policy
- Azure Resource Manager (ARM)
- Azure CLI / PowerShell (Cloud Shell)

---

## Related AZ-104 Exam Objectives

- Manage Azure Active Directory objects
- Manage role-based access control
- Manage subscriptions and governance
- Configure resource locks and policies

---

*Part of a 5-project AZ-104 hands-on portfolio | Built to demonstrate cloud administration skills for healthcare IT transition*