#  Lab 1: Azure Fundamentals & Resource Governance

##  Lab Objectives

By the end of this lab, participants will be able to:

* Understand how Microsoft Azure is structured globally
* Navigate Azure’s **management hierarchy**
* Create and manage **resource groups**
* Apply **naming conventions and tags**
* Assign **RBAC roles** using least-privilege
* Create and validate an **Azure Policy**
* Verify governance enforcement in Azure Portal

---

## Estimated Time

**~120 minutes**

---

## Prerequisites

* Azure account (Free Trial or corporate subscription)
* **Owner** or **User Access Administrator** role on the subscription
* Modern browser (Edge/Chrome recommended)

---

## Lab Scenario (Enterprise Context)

You are a **Cloud Engineer** onboarding a new application team.
Your responsibility is to:

* Organize Azure resources properly
* Enforce governance standards
* Control access using RBAC
* Ensure cost visibility and compliance from day one

---
![Image](https://dotnettrickscloud.blob.core.windows.net/img/azure/azure-infra.png)

![Image](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-setup-guide/media/organize-resources/scope-levels.png)

![Image](https://docs.azure.cn/en-us/role-based-access-control/media/overview/rbac-overview.png)

![Image](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/conditions-custom-security-attributes-example/role-assignment-condition.png)

![Image](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/shared/roles-constrained.png)

---

## Step 1: Sign in to Azure Portal

1. Open browser and navigate to
   👉 [https://portal.azure.com](https://portal.azure.com)
2. Sign in using your Azure credentials

✅ **Expected Result:**
You see the Azure Portal home dashboard

---

## Step 2: Explore Azure Global Infrastructure (Read-Only)

1. In the search bar, type **“Azure regions”**
2. Open **Azure Global Infrastructure** page
3. Observe:

   * Regions (East US, West Europe, Central India, etc.)
   * Paired regions
   * Availability Zones (AZ-1, AZ-2, AZ-3)

💡 **Trainer Note:**
Explain:

* Region = geographic area
* Availability Zone = isolated datacenter within a region
* Why zones improve **high availability**

---

##  Step 3: Understand Azure Management Hierarchy

Azure follows a strict hierarchy:

**Management Group → Subscription → Resource Group → Resource**

1. In portal search, open **Management Groups**
2. Review:

   * Root management group
   * Any child groups (if present)

💡 If using Free Trial, management groups may be limited—explain enterprise usage conceptually.

---

##  Step 4: Review Subscription Details

1. Navigate to **Subscriptions**
2. Click your active subscription
3. Observe:

   * Subscription ID
   * Cost management section
   * Access control (IAM)

✅ **Key Learning:**
Subscriptions are **billing + security boundaries**

---

##  Step 5: Create Resource Groups (Governance Foundation)

1. Go to **Resource Groups**
2. Click **Create**
3. Configure:

   * **Subscription:** Your active subscription
   * **Resource Group Name:**
     `rg-app-dev-centralindia`
   * **Region:** Central India
4. Click **Review + Create** → **Create**

Repeat to create:

* `rg-app-prod-centralindia`

💡 **Naming Convention Example:**

```
rg-<application>-<environment>-<region>
```

---

##  Step 6: Apply Tags for Cost & Ownership

1. Open **rg-app-dev-centralindia**
2. Select **Tags**
3. Add the following tags:

| Name        | Value      |
| ----------- | ---------- |
| environment | dev        |
| owner       | cloud-team |
| cost-center | training   |
| application | demo-app   |

4. Click **Save**

Repeat for **prod RG** with:

* `environment = prod`

✅ **Expected Result:**
Tags visible on resource group overview

---

##  Step 7: Understand Azure Identity & RBAC (Checkpoint)

Discuss concepts before action:

* Azure AD (Microsoft Entra ID)
* Users, Groups, Service Principals
* Role-Based Access Control (RBAC)
* Scope levels:

  * Subscription
  * Resource Group
  * Resource

Trainer checkpoint ✔️

---

##  Step 8: Assign Built-in RBAC Role (Developer)

1. Open **rg-app-dev-centralindia**
2. Select **Access control (IAM)**
3. Click **Add → Add role assignment**
4. Configure:

   * **Role:** Contributor
   * **Assign access to:** User
   * **Member:** Developer user
5. Click **Review + Assign**

✅ Developer can now manage resources **only in Dev RG**

---

##  Step 9: Assign Read-Only Role (Audit/User)

1. Open **rg-app-prod-centralindia**
2. Go to **Access control (IAM)**
3. Add role assignment:

   * **Role:** Reader
   * **Member:** Audit user
4. Assign

💡 **Least Privilege Principle:**
Never give Contributor/Owner unless required

---

##  Step 10: Built-in Roles vs Custom Roles (Conceptual)

Explain:

* Built-in roles (Owner, Contributor, Reader)
* Service-specific roles (VM Contributor, Network Contributor)
* Custom roles (compliance-driven scenarios)

⚠️ Custom role creation is **out of scope** for this foundation lab.

---

##  Step 11: Create Azure Policy – Enforce Mandatory Tags

1. Search **Policy**
2. Go to **Definitions**
3. Search for:
   **“Require a tag and its value on resources”**
4. Click **Assign**

Configure:

* **Scope:** Subscription
* **Policy definition:** Require a tag
* **Tag name:** `environment`
* **Effect:** Deny

Click **Review + Create**

---

##  Step 12: Validate Policy Enforcement

1. Try creating a new resource group:

   * Name: `rg-test-noncompliant`
   * **Do NOT add tags**
2. Attempt to create

❌ **Expected Result:**
Deployment fails due to policy violation

✅ Governance enforced successfully

---

##  Step 13: Review Policy Compliance

1. Go to **Policy → Compliance**
2. Review:

   * Non-compliant resources
   * Assigned policy
   * Scope and effect

💡 **Learning Point:**
Azure Policy enforces **preventive governance**

---

##  Step 14: Governance Summary Review

Confirm:

* Resource groups follow naming standard
* Tags applied consistently
* RBAC scoped correctly
* Policy preventing non-compliant resources

---

## ✅ Lab Validation Checklist

✔ Resource groups created
✔ Naming conventions applied
✔ Tags added and visible
✔ RBAC roles assigned correctly
✔ Least-privilege enforced
✔ Azure Policy assigned
✔ Policy violation validated

---

##  Key Takeaways

* Azure governance starts **before** resources are deployed
* Resource Groups are the core management unit
* RBAC controls *who can do what*
* Tags enable cost tracking and ownership
* Azure Policy enforces standards at scale
* Strong governance = secure + cost-controlled cloud
