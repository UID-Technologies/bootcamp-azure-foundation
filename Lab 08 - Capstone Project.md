# 🎓 Capstone Project: VM + VNet + Storage Deployment (Azure)

## Duration

**3 Hours (Hands-on only)**

---

##  Capstone Objective

Design, deploy, secure, monitor, and validate a **production-aware Azure infrastructure** consisting of:

* Secure **VNet + Subnets + NSGs**
* **Linux Virtual Machine** (no public IP)
* **Azure Storage** (Blob + File Share)
* **Monitoring, logging, and cost governance**
* **Infrastructure-as-Code (Bicep)** for at least one component

All work is performed in Microsoft Azure.

---

##  Scenario (Real-World)

You are onboarding a **new application workload** in Azure.
Your responsibility is to ensure:

* Secure network isolation
* Controlled access (RBAC + NSG)
* Centralized logging and monitoring
* Cost visibility with alerts
* Repeatable deployments
* Clear documentation for operations

---

##  Deliverables 

1. Azure resources deployed and validated
2. Architecture diagram
3. Bicep template(s)
4. Validation checklist
5. Short runbook (operations guide)

---

#  Architecture Overview


![Image](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/n-tier/images/single-vm-diagram.svg)

![Image](https://learn.microsoft.com/en-us/azure/architecture/networking/architecture/_images/hub-spoke.png)

![Image](https://learn.microsoft.com/en-us/azure/bastion/media/bastion-overview/architecture.png)

![Image](https://learn.microsoft.com/en-us/azure/bastion/media/bastion-nsg/figure-1.png)

![Image](https://imgopt.infoq.com/fit-in/3000x4000/filters%3Aquality%2885%29/filters%3Ano_upscale%28%29/news/2019/06/Azure-Bastion-VM/en/resources/2architecture-1561226707732.png)
---

**Components**

* Resource Group (governed, tagged)
* VNet with 3 subnets:

  * App Subnet
  * Management Subnet
  * AzureBastionSubnet
* NSG with restricted rules
* Linux VM (private IP only)
* Azure Bastion
* Storage Account

  * Blob container
  * File Share (mounted on VM)
* Log Analytics Workspace
* Budget + Cost Alert

---

#  PHASE 1: Design & Planning (30 Minutes)

## Step 1: Define Naming & Tagging Standard

Use this consistently:

```text
Resource Group: rg-capstone-dev-centralindia
VNet: vnet-capstone-dev
VM: vm-capstone-linux-01
Storage: stcapdev<unique>
```

**Mandatory Tags**

| Key         | Value      |
| ----------- | ---------- |
| environment | dev        |
| owner       | cloud-team |
| cost-center | capstone   |
| application | demo-app   |

---

## Step 2: Define Network Addressing

```text
VNet: 10.20.0.0/16
subnet-app: 10.20.1.0/24
subnet-mgmt: 10.20.2.0/24
AzureBastionSubnet: 10.20.10.0/27
```

---

#  PHASE 2: Build – Infrastructure Deployment (90 Minutes)

## Step 3: Create Resource Group

Azure Portal → Resource Groups → Create

* Name: `rg-capstone-dev-centralindia`
* Region: Central India
* Apply all mandatory tags

---

## Step 4: Create VNet & Subnets

Create VNet:

* Name: `vnet-capstone-dev`
* Address space: `10.20.0.0/16`

Add subnets:

* `subnet-app`
* `subnet-mgmt`
* `AzureBastionSubnet`

---

## Step 5: Create NSG & Rules

NSG Name: `nsg-capstone-app`

**Inbound Rules**

| Priority | Port | Source  | Action |
| -------- | ---- | ------- | ------ |
| 100      | 80   | Any     | Allow  |
| 110      | 22   | Bastion | Allow  |
| 4096     | Any  | Any     | Deny   |

Associate NSG with **subnet-app**

---

## Step 6: Deploy Azure Bastion

* Name: `bastion-capstone`
* VNet: `vnet-capstone-dev`
* Subnet: `AzureBastionSubnet`

---

## Step 7: Deploy Linux VM

* Name: `vm-capstone-linux-01`
* Image: Ubuntu 22.04 LTS
* Size: Standard B2s
* Public IP: **None**
* Subnet: `subnet-app`
* Authentication: SSH Key
* Enable **Boot Diagnostics**

### cloud-init

```yaml
#cloud-config
package_update: true
packages:
  - apache2
runcmd:
  - systemctl start apache2
  - echo "<h1>Capstone VM Running</h1>" > /var/www/html/index.html
```

---

## Step 8: Attach Data Disk

* Size: 32 GiB
* Type: Standard SSD

Inside VM:

```bash
sudo mkfs.ext4 /dev/sdc
sudo mkdir /mnt/data
sudo mount /dev/sdc /mnt/data
```

---

## Step 9: Create Storage Account

* Name: `stcapdev<unique>`
* Type: GPv2
* Redundancy: LRS
* Public access: Disabled

---

## Step 10: Configure Blob Storage

* Container: `app-data`
* Upload sample file
* Enable lifecycle:

  * Cool after 30 days
  * Delete after 365 days

---

## Step 11: Create Azure File Share

* Name: `appfiles`
* Quota: 100 GB

Mount on VM:

```bash
sudo apt install -y cifs-utils
sudo mkdir /mnt/appfiles
sudo mount -t cifs //<storage>.file.core.windows.net/appfiles /mnt/appfiles \
-o vers=3.0,username=<storage>,password=<key>
```

---

#  PHASE 3: Operability, Monitoring & Cost (45 Minutes)

## Step 12: Create Log Analytics Workspace

* Name: `law-capstone-dev`
* Region: Central India

---

## Step 13: Enable VM Insights

* Enable Insights on VM
* Connect to Log Analytics

---

## Step 14: Configure Diagnostic Settings

Send diagnostics to Log Analytics for:

* VM
* Storage Account

---

## Step 15: Create Budget & Alert

* Budget: Monthly
* Threshold: 80%
* Alert: Email

---

## Step 16: Review Azure Advisor

* Cost recommendations
* Security recommendations
* Reliability suggestions

---

#  PHASE 4: Infrastructure-as-Code (Bicep) (30 Minutes)

## Step 17: Create Bicep Template (Storage)

```bicep
param location string = resourceGroup().location
param storageName string

resource st 'Microsoft.Storage/storageAccounts@2022-09-01' = {
  name: storageName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}
```

Deploy:

```bash
az deployment group create \
  --resource-group rg-capstone-dev-centralindia \
  --template-file storage.bicep \
  --parameters storageName=stcapbicep$RANDOM
```

---

#  PHASE 5: Validation & Documentation (30 Minutes)

## Step 18: Validation Checklist

✔ VM reachable via Bastion
✔ Apache page accessible internally
✔ Storage mounted on VM
✔ Logs visible in Log Analytics
✔ Budget alert configured
✔ NSG rules enforced
✔ Bicep deployment successful

---

## Step 19: Architecture Diagram

Include:

* VNet & subnets
* VM
* NSG
* Bastion
* Storage
* Monitoring

---

## Step 20: Runbook (Short)

Include:

* How to access VM
* Where logs are stored
* How to check cost
* How to redeploy using Bicep

---

#  Evaluation Rubric (100%)

| Area                   | Weight |
| ---------------------- | ------ |
| Architecture & Design  | 20%    |
| Security & Governance  | 20%    |
| Implementation Quality | 30%    |
| Monitoring & Cost      | 20%    |
| Documentation          | 10%    |

---

##  Capstone Outcomes

After this capstone, learners can:

* Design Azure infrastructure end-to-end
* Apply enterprise governance
* Operate workloads confidently
* Use monitoring and cost controls
* Deploy repeatable infrastructure
* Think like a **Cloud Engineer**

