#  Lab 8: Capstone Project – End-to-End Azure Infrastructure

##  Lab Objectives

By the end of this capstone, participants will be able to:

* Design and deploy a **production-aware Azure infrastructure** from scratch
* Create a **secure VNet** with subnets and **NSGs**
* Deploy a **Linux VM** without public IP, accessed via **Azure Bastion**
* Configure **Azure Storage** (Blob + File Share) and mount on VM
* Enable **monitoring, logging, and cost governance**
* Deploy at least one component using **Infrastructure-as-Code (Bicep)**
* Validate the complete solution end-to-end

All work is performed in Microsoft Azure. This lab is **standalone** and does not require prior labs.

---

##  Estimated Time

**~180 minutes (3 hours)**

---

##  Prerequisites

* Azure account (Free Trial or corporate subscription)
* **Owner** or **Contributor** role on the subscription
* Modern browser (Edge/Chrome recommended)
* No prior Azure labs required—this capstone is self-contained

---

##  Lab Scenario (Enterprise Context)

You are a **Cloud Engineer** onboarding a new application workload for your organization.

**Your responsibility:**
* Design secure network isolation with controlled access
* Deploy a web server on a Linux VM with no direct internet exposure
* Integrate Azure Storage for application data and shared files
* Ensure visibility through monitoring and cost alerts
* Document the architecture and provide a runbook for operations

**Business requirements:**
* Application must be reachable internally (HTTP)
* Administrative access only via secure Bastion (no public IP on VM)
* Centralized logging for troubleshooting
* Cost visibility with budget alerts
* Repeatable deployment using IaC

---

##  Architecture Overview

![Image](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/n-tier/images/single-vm-diagram.svg)

**Components you will build:**

| Component | Purpose |
| --------- | ------- |
| Resource Group | Logical container with governance tags |
| VNet | Private network with 3 subnets |
| subnet-app | Application tier (VM runs here) |
| subnet-mgmt | Management tier (reserved for future use) |
| AzureBastionSubnet | Secure RDP/SSH access without public IP |
| NSG | Firewall rules (HTTP allowed, SSH via Bastion only) |
| Linux VM | Ubuntu with Apache, no public IP |
| Azure Bastion | Browser-based secure access to VM |
| Storage Account | Blob container + File Share |
| Log Analytics | Centralized logs and metrics |
| Budget + Alert | Cost governance |

---

#  PHASE 1: Design & Planning (30 Minutes)

---

##  Step 1: Define Naming and Tagging Standard

**Purpose:** Consistent naming and tagging enable cost tracking, governance, and operations at scale.

1. Open a text editor or notes—you will use these values throughout the lab
2. Adopt this naming convention:

| Resource Type | Name | Example |
| ------------- | ---- | ------- |
| Resource Group | `rg-capstone-dev-centralindia` | Logical container |
| Virtual Network | `vnet-capstone-dev` | Network boundary |
| Subnet (App) | `subnet-app` | Application tier |
| Subnet (Mgmt) | `subnet-mgmt` | Management tier |
| NSG | `nsg-capstone-app` | Firewall rules |
| Bastion | `bastion-capstone` | Secure access |
| VM | `vm-capstone-linux-01` | Compute |
| Storage | `stcapdev<unique>` | Must be globally unique (e.g., `stcapdev12345`) |

3. Define **mandatory tags** for the Resource Group:

| Key | Value |
| --- | ----- |
| environment | dev |
| owner | cloud-team |
| cost-center | capstone |
| application | demo-app |

💡 **Learning Point:** Tags enable cost allocation, compliance, and automation. Use them consistently.

---

##  Step 2: Define Network Addressing

**Purpose:** Proper IP planning prevents conflicts and supports future growth.

Document these values:

```
VNet address space:     10.20.0.0/16
subnet-app:             10.20.1.0/24   (Application VM)
subnet-mgmt:            10.20.2.0/24   (Management - reserved)
AzureBastionSubnet:     10.20.10.0/27  (Bastion - /27 required)
```

💡 **Learning Point:** Azure Bastion requires a dedicated subnet named exactly `AzureBastionSubnet` with minimum /27.

---

#  PHASE 2: Infrastructure Deployment (90 Minutes)

---

##  Step 3: Create Resource Group with Tags

**Purpose:** Resource groups are the primary management and billing boundary in Azure.

1. Sign in to [Azure Portal](https://portal.azure.com)
2. Search **Resource Groups** in the top search bar
3. Click **+ Create**
4. Configure:
   * **Subscription:** Your active subscription
   * **Resource group name:** `rg-capstone-dev-centralindia`
   * **Region:** Central India
5. Click **Review + Create** → **Create**
6. Open the new resource group → **Tags**
7. Add tags:

   | Name | Value |
   | ---- | ----- |
   | environment | dev |
   | owner | cloud-team |
   | cost-center | capstone |
   | application | demo-app |
8. Click **Save**

✅ **Expected Result:** Resource group created with all tags visible in Overview.

---

##  Step 4: Create Virtual Network and Subnets

**Purpose:** VNet provides isolated network space. Subnets segment workloads for security and routing.

1. Search **Virtual networks** → **+ Create**
2. **Basics** tab:
   * **Subscription:** Your subscription
   * **Resource group:** `rg-capstone-dev-centralindia`
   * **Name:** `vnet-capstone-dev`
   * **Region:** Central India
3. Click **Next: IP Addresses**
4. **IPv4 address space:** `10.20.0.0/16`
5. Under **Subnets**, click **+ Add subnet**:
   * **Subnet name:** `subnet-app`
   * **Subnet address range:** `10.20.1.0/24`
   * Click **Add**
6. Add second subnet:
   * **Subnet name:** `subnet-mgmt`
   * **Subnet address range:** `10.20.2.0/24`
   * Click **Add**
7. Add third subnet:
   * **Subnet name:** `AzureBastionSubnet`
   * **Subnet address range:** `10.20.10.0/27`
   * Click **Add**
8. Click **Review + Create** → **Create**

✅ **Expected Result:** VNet created with three subnets. AzureBastionSubnet is reserved for Bastion.

---

##  Step 5: Create Network Security Group and Rules

**Purpose:** NSG acts as a virtual firewall to control inbound and outbound traffic.

1. Search **Network security groups** → **+ Create**
2. Configure:
   * **Subscription:** Your subscription
   * **Resource group:** `rg-capstone-dev-centralindia`
   * **Name:** `nsg-capstone-app`
   * **Region:** Central India
3. Click **Review + Create** → **Create**
4. Open `nsg-capstone-app` → **Inbound security rules**
5. Click **+ Add** and create:

   **Rule 1 – Allow HTTP (port 80):**
   * **Source:** Any
   * **Source port ranges:** *
   * **Destination:** Any
   * **Destination port ranges:** 80
   * **Protocol:** TCP
   * **Action:** Allow
   * **Priority:** 100
   * **Name:** `allow-http`
   * Click **Add**

   **Rule 2 – Allow SSH from Bastion subnet:**
   * **Source:** VirtualNetwork
   * **Source port ranges:** *
   * **Destination:** Any
   * **Destination port ranges:** 22
   * **Protocol:** TCP
   * **Action:** Allow
   * **Priority:** 110
   * **Name:** `allow-ssh-from-bastion`
   * Click **Add**

6. Associate NSG with subnet:
   * Open `nsg-capstone-app` → **Subnets**
   * Click **+ Associate**
   * **Virtual network:** `vnet-capstone-dev`
   * **Subnet:** `subnet-app`
   * Click **OK**

✅ **Expected Result:** NSG created with two inbound rules, associated with subnet-app.

---

##  Step 6: Deploy Azure Bastion

**Purpose:** Bastion provides secure RDP/SSH access without exposing the VM to the internet.

1. Search **Bastions** → **+ Create**
2. Configure:
   * **Subscription:** Your subscription
   * **Resource group:** `rg-capstone-dev-centralindia`
   * **Name:** `bastion-capstone`
   * **Region:** Central India
   * **Virtual network:** `vnet-capstone-dev`
   * **Subnet:** `AzureBastionSubnet` (10.20.10.0/27)
   * **Tier:** Basic (for lab cost savings)
3. Click **Review + Create** → **Create**

⏳ **Note:** Bastion deployment takes approximately 5–10 minutes.

✅ **Expected Result:** Bastion resource shows "Succeeded" status.

---

##  Step 7: Deploy Linux Virtual Machine

**Purpose:** VM hosts the sample web application. No public IP keeps it secure; access is via Bastion only.

1. Search **Virtual machines** → **+ Create** → **Azure virtual machine**
2. **Basics** tab:
   * **Subscription:** Your subscription
   * **Resource group:** `rg-capstone-dev-centralindia`
   * **Virtual machine name:** `vm-capstone-linux-01`
   * **Region:** Central India
   * **Availability options:** No infrastructure redundancy required
   * **Image:** Ubuntu Server 22.04 LTS
   * **Size:** Standard B2s
3. **Administrator account:**
   * **Authentication type:** SSH public key
   * **Username:** `azureuser`
   * **SSH public key source:** Generate new key pair
   * **Key pair name:** `vm-capstone-key`
   * ⬇️ **Download the private key** and store it safely
4. **Inbound port rules:** Select **None** (no public inbound ports)
5. Click **Next: Disks**
   * **OS disk type:** Premium SSD (or Standard SSD for cost)
   * Click **Next: Networking**
6. **Networking:**
   * **Virtual network:** `vnet-capstone-dev`
   * **Subnet:** `subnet-app`
   * **Public IP:** None
   * **NIC network security group:** None (NSG already on subnet)
7. Click **Next: Management**
   * **Boot diagnostics:** Enable with managed storage
8. Click **Next: Advanced**
   * Under **Custom data**, paste this **cloud-init** script:

```yaml
#cloud-config
package_update: true
packages:
  - apache2
runcmd:
  - systemctl start apache2
  - systemctl enable apache2
  - echo "<h1>Capstone VM Running - Lab 08</h1><p>Welcome to Azure!</p>" > /var/www/html/index.html
```

9. Click **Review + Create** → **Create**

✅ **Expected Result:** VM deploys. Apache installs automatically on first boot. Wait 2–3 minutes for cloud-init to complete.

---

##  Step 8: Connect to VM via Bastion and Validate Apache

**Purpose:** Verify secure access and that the web server is running.

1. Open **vm-capstone-linux-01** → **Connect**
2. Select **Bastion** tab
3. **Username:** `azureuser`
4. **Authentication type:** SSH Private Key
5. **Private Key:** Paste contents of `vm-capstone-key` (downloaded in Step 7)
6. Click **Connect**
7. In the Bastion session, run:

```bash
systemctl status apache2
curl http://localhost
```

✅ **Expected Result:** Apache is active; curl returns the HTML page.

---

##  Step 9: Attach and Mount Data Disk

**Purpose:** Separate application data from OS disk for durability and scalability.

1. Open **vm-capstone-linux-01** → **Disks**
2. Click **+ Create and attach a new disk**
3. Configure:
   * **Name:** `vm-capstone-data01`
   * **Size:** 32 GiB
   * **Type:** Standard SSD
4. Click **Save**
5. Connect to VM via Bastion (Step 8)
6. Run:

```bash
lsblk
sudo mkfs.ext4 /dev/sdc
sudo mkdir -p /mnt/data
sudo mount /dev/sdc /mnt/data
df -h
```

✅ **Expected Result:** Data disk mounted at `/mnt/data`.

---

##  Step 10: Create Storage Account

**Purpose:** Storage provides Blob and File Share for application data and shared files.

1. Search **Storage accounts** → **+ Create**
2. **Basics:**
   * **Subscription:** Your subscription
   * **Resource group:** `rg-capstone-dev-centralindia`
   * **Storage account name:** `stcapdev` + unique suffix (e.g., `stcapdev12345`)
   * **Region:** Central India
   * **Performance:** Standard
   * **Redundancy:** LRS
3. **Advanced** tab:
   * **Allow Blob public access:** Disabled
4. Click **Review + Create** → **Create**

✅ **Expected Result:** Storage account created. Note the name for later steps.

---

##  Step 11: Create Blob Container and Upload Sample File

**Purpose:** Blob storage for unstructured data (logs, backups, artifacts).

1. Open your storage account → **Containers**
2. Click **+ Container**
3. **Name:** `app-data`
4. **Public access level:** Private
5. Click **Create**
6. Open `app-data` → **Upload**
7. Upload any small file (e.g., a text file with "Hello from Capstone")
8. **Lifecycle management** (optional): Storage account → **Data management** → **Lifecycle management** → Add rule to move blobs to Cool tier after 30 days

✅ **Expected Result:** Blob container with at least one file.

---

##  Step 12: Create Azure File Share and Mount on VM

**Purpose:** Shared file storage accessible from the VM (e.g., for application config or logs).

1. Open storage account → **File shares**
2. Click **+ File share**
3. **Name:** `appfiles`
4. **Quota:** 100 GiB
5. Click **Create**
6. Open `appfiles` → **Connect**
7. Select **Linux** and copy the mount command
8. Connect to VM via Bastion
9. Install CIFS and mount:

```bash
sudo apt-get update
sudo apt-get install -y cifs-utils
sudo mkdir -p /mnt/appfiles
```

10. Run the mount command from the portal (replace `<storage-account-name>` and `<storage-account-key>`):

```bash
sudo mount -t cifs //<storage-account-name>.file.core.windows.net/appfiles /mnt/appfiles \
  -o vers=3.0,username=<storage-account-name>,password=<storage-account-key>,dir_mode=0777,file_mode=0777
```

11. Validate:

```bash
echo "Hello from Azure Files" | sudo tee /mnt/appfiles/test.txt
ls -l /mnt/appfiles
```

✅ **Expected Result:** File share mounted; test.txt visible in Azure Portal under File share.

---

#  PHASE 3: Monitoring & Cost Governance (45 Minutes)

---

##  Step 13: Create Log Analytics Workspace

**Purpose:** Centralized log and metric collection for troubleshooting and compliance.

1. Search **Log Analytics workspaces** → **+ Create**
2. Configure:
   * **Subscription:** Your subscription
   * **Resource group:** `rg-capstone-dev-centralindia`
   * **Name:** `law-capstone-dev`
   * **Region:** Central India
3. Click **Review + Create** → **Create**

✅ **Expected Result:** Log Analytics workspace created.

---

##  Step 14: Enable VM Insights and Diagnostic Settings

**Purpose:** Collect VM metrics and logs for monitoring and alerting.

1. Open **vm-capstone-linux-01** → **Insights**
2. Click **Enable**
3. Select Log Analytics workspace: `law-capstone-dev`
4. Click **Configure**
5. Open **vm-capstone-linux-01** → **Diagnostic settings**
6. Click **+ Add diagnostic setting**
7. **Name:** `vm-diag-to-law`
8. **Destination:** Send to Log Analytics workspace → `law-capstone-dev`
9. Enable **All logs** and **All metrics**
10. Click **Save**

✅ **Expected Result:** VM sends logs and metrics to Log Analytics.

---

##  Step 15: Configure Diagnostic Settings for Storage Account

**Purpose:** Track storage transactions and access for auditing.

1. Open your storage account → **Diagnostic settings**
2. Click **+ Add diagnostic setting**
3. **Name:** `storage-diag-to-law`
4. **Destination:** Log Analytics workspace → `law-capstone-dev`
5. **Logs:** Blob, File
6. **Metrics:** Transaction
7. Click **Save**

---

##  Step 16: Create Budget and Cost Alert

**Purpose:** Avoid billing surprises with proactive alerts.

1. Search **Cost Management + Billing**
2. Under **Cost Management**, click **Budgets**
3. Click **+ Add**
4. **Scope:** Subscription
5. **Budget name:** `capstone-monthly-budget`
6. **Reset period:** Monthly
7. **Amount:** 500 (or your preferred limit in local currency)
8. **Budget alerts:**
   * At 80% → Email
   * At 100% → Email
9. **Email recipients:** Your email
10. Click **Create**

✅ **Expected Result:** Budget created; you will receive email when thresholds are reached.

---

##  Step 17: Review Azure Advisor

**Purpose:** Get Azure best-practice recommendations.

1. Search **Advisor**
2. Review **Cost**, **Security**, **Reliability**, **Performance**
3. Note any recommendations applicable to your resources

---

#  PHASE 4: Infrastructure-as-Code (30 Minutes)

---

##  Step 18: Deploy Storage Account Using Bicep

**Purpose:** Demonstrate repeatable deployment using IaC.

1. Open **Cloud Shell** (Bash)
2. Create Bicep file:

```bash
cd ~
cat > storage.bicep << 'EOF'
param location string = resourceGroup().location
param storageName string

resource st 'Microsoft.Storage/storageAccounts@2022-09-01' = {
  name: storageName
  location: location
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
}
EOF
```

3. Deploy:

```bash
az deployment group create \
  --resource-group rg-capstone-dev-centralindia \
  --template-file storage.bicep \
  --parameters storageName=stcapbicep$RANDOM
```

✅ **Expected Result:** New storage account deployed via Bicep. Verify in Resource Group.

---

#  PHASE 5: Validation & Documentation (30 Minutes)

---

##  Step 19: Complete Validation Checklist

**Purpose:** Confirm all components work as designed.

| Check | How to Validate |
| ----- | ---------------- |
| ✔ VM reachable via Bastion | Connect → Bastion → SSH succeeds |
| ✔ Apache accessible | From VM: `curl http://localhost` |
| ✔ Data disk mounted | From VM: `df -h` shows /mnt/data |
| ✔ File share mounted | From VM: `ls /mnt/appfiles` |
| ✔ Logs in Log Analytics | Log Analytics → Logs → Run: `Heartbeat \| take 10` |
| ✔ Budget configured | Cost Management → Budgets |
| ✔ NSG rules active | NSG → Inbound rules |
| ✔ Bicep deployment | Resource Group → Deployments |

---

##  Step 20: Create Architecture Diagram and Runbook

**Purpose:** Document the solution for operations and handover.

**Architecture diagram** (draw or use tool) should include:
* VNet and subnets
* VM, Bastion, NSG
* Storage Account
* Log Analytics
* Data flow (Bastion → VM, VM → Storage)

**Short runbook** should include:
* How to access the VM (Bastion steps)
* Where logs are stored (Log Analytics workspace)
* How to check cost (Cost Management)
* How to redeploy storage via Bicep

---

##  Lab Validation Checklist

✔ Resource group created with tags
✔ VNet and subnets configured
✔ NSG created and associated
✔ Bastion deployed
✔ Linux VM deployed (no public IP)
✔ Apache running (cloud-init)
✔ Data disk attached and mounted
✔ Storage account created
✔ Blob container and File Share configured
✔ File share mounted on VM
✔ Log Analytics workspace created
✔ VM and Storage diagnostics configured
✔ Budget and alert created
✔ Bicep deployment successful
✔ Architecture diagram and runbook completed

---

##  Key Takeaways

* End-to-end Azure infrastructure can be built in a few hours
* Security by design: no public IP on VM, Bastion for access
* Governance: tags, NSGs, and cost alerts from day one
* Monitoring: Log Analytics centralizes logs and metrics
* IaC: Bicep enables repeatable, auditable deployments
* Documentation (runbook) is essential for operations
