#  Lab: Security, Monitoring, Cost Management & IaC (Step-by-Step)

##  Lab Objectives

By the end of this lab, learners will be able to:

* Enable and use **Azure Monitor** for metrics and logs
* Create and use a **Log Analytics Workspace**
* Configure **diagnostic settings** for Virtual Machines and Storage Accounts
* Understand and use **Azure Advisor** recommendations
* Set up **Cost Management budgets, alerts, and cost analysis**
* Understand **Managed Identities** and **Azure Key Vault** (security foundations)
* Understand **ARM vs Bicep**
* Deploy a resource using a **simple Bicep template**

All tasks are performed in Microsoft Azure.

---

##  Estimated Time

**~120 minutes (2 Hours)**

---

##  Prerequisites

* Completed Azure labs for:

  * Resource Governance
  * VNet & NSG
  * Virtual Machines
  * Storage
* Existing resources:

  * Resource Group: `rg-app-dev-centralindia`
  * VM: `vm-app-linux-01`
  * Storage Account: `stappdev<uniqueid>`
* Contributor access on subscription
* Azure Portal + Cloud Shell access

---

##  Lab Scenario (Enterprise Context)

Your application is live.
Now you must ensure:

* **Visibility** into performance and failures
* **Security** without secrets in code
* **Cost control** with alerts before overruns
* **Repeatable deployments** using Infrastructure-as-Code

----

![Image](https://learn.microsoft.com/en-us/azure/azure-monitor/fundamentals/media/enterprise-monitoring-architecture/architecture-diagram.png)

![Image](https://learn.microsoft.com/en-us/power-bi/transform-model/log-analytics/media/desktop-log-analytics-overview/log-analytics-01.png)

![Image](https://learn.microsoft.com/en-us/azure/azure-monitor/agents/media/diagnostics-extension-windows-install/enable-monitoring.png)

![Image](https://learn.microsoft.com/en-us/azure/virtual-machines/media/boot-diagnostics/boot-diagnostics-enable-portal.png)

![Image](https://pitstop.manageengine.com/galleryDocuments/edbsn917d81196909f4b481575514fa429e996ba9bf54854df762bed8b347d24317041f18b70a806e08933b8a84c3467e592d096af060125c6d747ee8f917c703d1e8?inline=true)



---

##  Step 1: Azure Monitor – Concepts Checkpoint

Before configuration, confirm understanding:

* **Metrics**

  * Near real-time numeric data (CPU, memory, disk, network)
* **Logs**

  * Detailed events and diagnostics
* **Azure Monitor**

  * Central observability platform for Azure

Trainer checkpoint ✔️

---

##  Step 2: Create a Log Analytics Workspace

1. In Azure Portal, search **Log Analytics workspaces**
2. Click **Create**
3. Configure:

   * **Subscription:** Active subscription
   * **Resource Group:** `rg-app-dev-centralindia`
   * **Name:** `law-app-dev`
   * **Region:** Central India
4. Click **Review + Create → Create**

✅ Workspace created successfully

---

##  Step 3: Enable VM Insights (Monitoring)

1. Open **vm-app-linux-01**
2. Select **Insights**
3. Click **Enable**
4. Choose:

   * Log Analytics Workspace: `law-app-dev`
5. Confirm and enable

💡 **What this enables:**

* CPU, memory, disk, network monitoring
* Dependency and performance visibility

---

##  Step 4: Explore Azure Monitor Metrics

1. Open VM → **Monitoring → Metrics**
2. Select metrics:

   * CPU Percentage
   * Disk Read Operations/sec
   * Network In Total
3. Adjust:

   * Time range
   * Aggregation (Avg / Max)

✅ You can now visually monitor VM health

---

##  Step 5: Configure Diagnostic Settings for VM

1. Open **vm-app-linux-01**
2. Go to **Diagnostic settings**
3. Click **Add diagnostic setting**
4. Configure:

   * **Name:** `vm-diag-to-law`
   * Destination: **Send to Log Analytics workspace**
   * Select:

     * All logs
     * All metrics
5. Click **Save**

💡 Logs are now centralized for analysis.

---

##  Step 6: Configure Diagnostic Settings for Storage Account

1. Open storage account → **Diagnostic settings**
2. Click **Add diagnostic setting**
3. Configure:

   * **Name:** `storage-diag-to-law`
   * Destination: Log Analytics Workspace
   * Logs:

     * Blob
     * File
     * Queue
   * Metrics:

     * Transaction
4. Save

---

## Step 7: Query Logs Using Log Analytics (KQL)

1. Open **Log Analytics Workspace → Logs**
2. Run basic queries:

**Azure Activity Logs**

```kusto
AzureActivity
| sort by TimeGenerated desc
```

**VM Performance Logs**

```kusto
Perf
| where ObjectName == "Processor"
| limit 10
```

💡 **Learning Point:**
KQL is the foundation of troubleshooting and observability.

---

##  Step 8: Azure Advisor Overview (Read-Only)

1. Search **Advisor**
2. Review categories:

   * Cost
   * Security
   * Reliability
   * Performance
3. Open any recommendation

💡 Advisor provides **best-practice guidance**, not enforcement.

---

##  Step 9: Cost Management – Create a Budget

1. Search **Cost Management**
2. Go to **Budgets**
3. Click **Add**
4. Configure:

   * **Scope:** Subscription
   * **Budget name:** `dev-monthly-budget`
   * **Amount:** 1000 (or suitable value)
   * **Reset period:** Monthly
5. Alerts:

   * 80% actual spend → Email notification
6. Click **Create**

✅ Cost alert configured

---

##  Step 10: Cost Analysis & Tag-Based Views

1. Go to **Cost Management → Cost analysis**
2. Group by:

   * Resource Group
   * Service Name
   * Tag (environment)
3. Observe:

   * VM cost
   * Storage cost

💡 This is how enterprises **control and optimize spend**.

---

##  Step 11: Security Foundations – Managed Identity (Conceptual)

Explain:

* **System-assigned managed identity**
* **User-assigned managed identity**
* No secrets or credentials in code
* Used with:

  * Storage
  * Key Vault
  * Azure APIs

(No assignment required in this foundation lab)

---

##  Step 12: Azure Key Vault Overview (Conceptual)

Discuss:

* Secure storage for:

  * Secrets
  * Keys
  * Certificates
* Integration with Managed Identity
* Centralized secret rotation

💡 **Key Rule:**
Applications should **never hardcode secrets**.

---

##  Step 13: Infrastructure-as-Code – ARM vs Bicep

Conceptual comparison:

| ARM Templates    | Bicep            |
| ---------------- | ---------------- |
| JSON             | Simple DSL       |
| Verbose          | Clean & readable |
| Hard to maintain | Easy to version  |
| Native           | Compiles to ARM  |

👉 **Bicep is the recommended approach**

---

##  Step 14: Create a Simple Bicep Template

1. Open **Cloud Shell**
2. Select **Bash**
3. Create a file:

```bash
nano storage.bicep
```

Paste:

```bicep
param location string = resourceGroup().location
param storageName string

resource storage 'Microsoft.Storage/storageAccounts@2022-09-01' = {
  name: storageName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}
```

Save and exit.

---

##  Step 15: Deploy Resource Using Bicep

Run:

```bash
az deployment group create \
  --resource-group rg-app-dev-centralindia \
  --template-file storage.bicep \
  --parameters storageName=stbicepdev$RANDOM
```

✅ Storage account deployed via **IaC**

---

##  Step 16: Validate Deployment

1. Open **Resource Group**
2. Confirm:

   * New storage account exists
   * Deployment succeeded
3. Review **Deployments** section

---

##  Step 17: Operational Readiness Review

Confirm:

* Monitoring enabled
* Logs centralized
* Diagnostics configured
* Budget and alerts active
* IaC deployment successful

This completes **operability + governance**.

---

##  Lab Validation Checklist

✔ Log Analytics workspace created
✔ VM Insights enabled
✔ VM & Storage diagnostics configured
✔ Logs queried successfully
✔ Azure Advisor reviewed
✔ Cost budget & alert created
✔ Cost analysis explored
✔ Bicep template deployed

---

##  Key Takeaways

* Monitoring prevents blind failures
* Logs explain *why* issues happen
* Cost alerts avoid billing surprises
* Managed Identity removes secrets
* Key Vault centralizes security
* Bicep enables repeatable, auditable deployments
