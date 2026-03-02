#  Lab 9: Capstone Project – VNet with Public & Private Subnets

##  Lab Objectives

By the end of this capstone, participants will be able to:

* Design a **VNet with public and private subnets**
* Deploy a **sample web application** on a VM in the **public subnet**
* Deploy a **backend VM** in the **private subnet** (no public IP)
* Configure **NSGs** to control traffic between tiers
* Use **Azure Bastion** for secure access to the private VM
* Validate **internet access** to the web app and **internal-only** access to the backend
* Understand **DMZ-style** network segmentation

All work is performed in Microsoft Azure. This lab is **standalone** and does not require prior labs.

---

##  Estimated Time

**~150 minutes (2.5 hours)**

---

##  Prerequisites

* Azure account (Free Trial or corporate subscription)
* **Owner** or **Contributor** role on the subscription
* Modern browser (Edge/Chrome recommended)

---

##  Lab Scenario (Enterprise Context)

You are a **Cloud Engineer** building a **two-tier web application** for your organization.

**Architecture requirements:**
* **Public subnet:** Web tier VM with a sample app (Apache) accessible from the internet
* **Private subnet:** Backend VM with no public IP—only reachable from the web tier or via Bastion
* **Network segmentation:** Traffic flows from internet → public subnet → private subnet (no direct internet to private)

**Business context:**
* Web tier serves static content and forwards API calls to the backend
* Backend holds sensitive logic and data—must not be exposed to the internet
* Administrative access to both VMs: Bastion for private VM; SSH/RDP for public VM (or Bastion for both)

---

##  Architecture Overview

```
                    Internet
                        │
                        ▼
              ┌─────────────────────┐
              │   Public Subnet     │
              │  (10.30.1.0/24)     │
              │                     │
              │  vm-web-01          │
              │  • Public IP        │
              │  • Apache (port 80) │
              │  • Sample web app   │
              └──────────┬──────────┘
                         │
                         │ Internal only
                         ▼
              ┌─────────────────────┐
              │   Private Subnet    │
              │  (10.30.2.0/24)     │
              │                     │
              │  vm-backend-01      │
              │  • No Public IP     │
              │  • API / Data       │
              │  • Bastion access   │
              └─────────────────────┘
                         │
                         │ Bastion
                         ▼
              ┌─────────────────────┐
              │ AzureBastionSubnet  │
              │  (10.30.10.0/27)    │
              └─────────────────────┘
```

**Components:**

| Component | Subnet | Public IP | Purpose |
| --------- | ------ | --------- | ------- |
| vm-web-01 | Public | Yes | Web tier – sample app, internet-facing |
| vm-backend-01 | Private | No | Backend tier – internal only |
| bastion-capstone09 | AzureBastionSubnet | Yes (Bastion) | Secure admin access |
| NSG (public) | Public subnet | - | Allow HTTP/HTTPS from internet, SSH from Bastion |
| NSG (private) | Private subnet | - | Allow traffic from public subnet only |

---

#  PHASE 1: Design & Planning (20 Minutes)

---

##  Step 1: Define Naming and Network Addressing

**Purpose:** Consistent naming and IP planning support clarity and future expansion.

**Naming convention:**

| Resource | Name |
| -------- | ---- |
| Resource Group | `rg-capstone09-dev-centralindia` |
| VNet | `vnet-capstone09-dev` |
| Public Subnet | `subnet-public` |
| Private Subnet | `subnet-private` |
| NSG (Public) | `nsg-capstone09-public` |
| NSG (Private) | `nsg-capstone09-private` |
| Web VM | `vm-web-01` |
| Backend VM | `vm-backend-01` |
| Bastion | `bastion-capstone09` |

**Network addressing:**

```
VNet:                10.30.0.0/16
subnet-public:       10.30.1.0/24   (Web tier)
subnet-private:      10.30.2.0/24   (Backend tier)
AzureBastionSubnet:  10.30.10.0/27  (Bastion)
```

**Tags:**

| Key | Value |
| --- | ----- |
| environment | dev |
| owner | cloud-team |
| application | capstone09 |

---

#  PHASE 2: Network Infrastructure (45 Minutes)

---

##  Step 2: Create Resource Group

**Purpose:** Logical container for all capstone resources with governance tags.

1. Sign in to [Azure Portal](https://portal.azure.com)
2. Search **Resource Groups** → **+ Create**
3. Configure:
   * **Subscription:** Your active subscription
   * **Resource group name:** `rg-capstone09-dev-centralindia`
   * **Region:** Central India
4. Click **Review + Create** → **Create**
5. Open the resource group → **Tags**
6. Add: `environment` = `dev`, `owner` = `cloud-team`, `application` = `capstone09`
7. Click **Save**

✅ **Expected Result:** Resource group created with tags.

---

##  Step 3: Create Virtual Network with Public and Private Subnets

**Purpose:** VNet with separate subnets for public-facing and internal-only workloads.

1. Search **Virtual networks** → **+ Create**
2. **Basics:**
   * **Subscription:** Your subscription
   * **Resource group:** `rg-capstone09-dev-centralindia`
   * **Name:** `vnet-capstone09-dev`
   * **Region:** Central India
3. Click **Next: IP Addresses**
4. **IPv4 address space:** `10.30.0.0/16`
5. Add subnets:

   **Subnet 1 – Public:**
   * **Subnet name:** `subnet-public`
   * **Subnet address range:** `10.30.1.0/24`
   * Click **Add**

   **Subnet 2 – Private:**
   * **Subnet name:** `subnet-private`
   * **Subnet address range:** `10.30.2.0/24`
   * Click **Add**

   **Subnet 3 – Bastion:**
   * **Subnet name:** `AzureBastionSubnet`
   * **Subnet address range:** `10.30.10.0/27`
   * Click **Add**

6. Click **Review + Create** → **Create**

✅ **Expected Result:** VNet with three subnets. Public and private subnets are clearly separated.

---

##  Step 4: Create NSG for Public Subnet

**Purpose:** Allow HTTP from internet for the web app; allow SSH from Bastion for administration.

1. Search **Network security groups** → **+ Create**
2. Configure:
   * **Resource group:** `rg-capstone09-dev-centralindia`
   * **Name:** `nsg-capstone09-public`
   * **Region:** Central India
3. Click **Review + Create** → **Create**
4. Open `nsg-capstone09-public` → **Inbound security rules**
5. Add rules:

   **Allow HTTP (port 80):**
   * **Source:** Any
   * **Source port ranges:** *
   * **Destination:** Any
   * **Destination port ranges:** 80
   * **Protocol:** TCP
   * **Action:** Allow
   * **Priority:** 100
   * **Name:** `allow-http-internet`
   * Click **Add**

   **Allow SSH from Bastion (VNet):**
   * **Source:** VirtualNetwork
   * **Destination port ranges:** 22
   * **Protocol:** TCP
   * **Action:** Allow
   * **Priority:** 110
   * **Name:** `allow-ssh-from-bastion`
   * Click **Add**

6. Associate with subnet:
   * **Subnets** → **+ Associate**
   * **Virtual network:** `vnet-capstone09-dev`
   * **Subnet:** `subnet-public`
   * Click **OK**

✅ **Expected Result:** Public subnet allows HTTP from internet and SSH from VNet (Bastion).

---

##  Step 5: Create NSG for Private Subnet

**Purpose:** Allow traffic only from the public subnet (web tier) and from Bastion. No internet access.

1. Search **Network security groups** → **+ Create**
2. Configure:
   * **Resource group:** `rg-capstone09-dev-centralindia`
   * **Name:** `nsg-capstone09-private`
   * **Region:** Central India
3. Click **Review + Create** → **Create**
4. Open `nsg-capstone09-private` → **Inbound security rules**
5. Add rules:

   **Allow from Public Subnet (internal traffic):**
   * **Source:** VirtualNetwork
   * **Source port ranges:** *
   * **Destination:** Any
   * **Destination port ranges:** 22, 80, 443
   * **Protocol:** TCP
   * **Action:** Allow
   * **Priority:** 100
   * **Name:** `allow-from-public-subnet`
   * Click **Add**

   **Allow SSH from Bastion:**
   * **Source:** VirtualNetwork
   * **Destination port ranges:** 22
   * **Protocol:** TCP
   * **Action:** Allow
   * **Priority:** 110
   * **Name:** `allow-ssh-from-bastion`
   * Click **Add**

6. Associate with subnet:
   * **Subnets** → **+ Associate**
   * **Virtual network:** `vnet-capstone09-dev`
   * **Subnet:** `subnet-private`
   * Click **OK**

✅ **Expected Result:** Private subnet accepts traffic only from the VNet (public subnet or Bastion). No direct internet access.

---

##  Step 6: Deploy Azure Bastion

**Purpose:** Secure administrative access to both VMs without exposing RDP/SSH to the internet.

1. Search **Bastions** → **+ Create**
2. Configure:
   * **Resource group:** `rg-capstone09-dev-centralindia`
   * **Name:** `bastion-capstone09`
   * **Region:** Central India
   * **Virtual network:** `vnet-capstone09-dev`
   * **Subnet:** `AzureBastionSubnet`
   * **Tier:** Basic
3. Click **Review + Create** → **Create**

⏳ Bastion deployment takes ~5–10 minutes.

---

#  PHASE 3: Deploy VMs and Sample Application (60 Minutes)

---

##  Step 7: Deploy Web Tier VM (Public Subnet)

**Purpose:** VM in public subnet with public IP to host the sample web app, reachable from the internet.

1. Search **Virtual machines** → **+ Create** → **Azure virtual machine**
2. **Basics:**
   * **Resource group:** `rg-capstone09-dev-centralindia`
   * **VM name:** `vm-web-01`
   * **Region:** Central India
   * **Image:** Ubuntu Server 22.04 LTS
   * **Size:** Standard B2s
3. **Administrator account:**
   * **Authentication type:** SSH public key
   * **Username:** `azureuser`
   * **SSH public key source:** Generate new key pair
   * **Key pair name:** `vm-web-key`
   * ⬇️ Download and save the private key
4. **Inbound port rules:** Allow selected ports → **SSH (22)**, **HTTP (80)**
5. **Next: Disks** → Defaults
6. **Next: Networking:**
   * **Virtual network:** `vnet-capstone09-dev`
   * **Subnet:** `subnet-public`
   * **Public IP:** Create new → `vm-web-01-ip`
   * **NIC NSG:** None (NSG on subnet)
7. **Next: Management** → Boot diagnostics: Enable
8. **Next: Advanced** → **Custom data**, paste:

```yaml
#cloud-config
package_update: true
packages:
  - apache2
runcmd:
  - systemctl start apache2
  - systemctl enable apache2
  - echo '<h1>Lab 09 - Public Subnet Web App</h1><p>Welcome! This VM is in the public subnet.</p>' > /var/www/html/index.html
  - echo '<h2>Backend Status</h2><p>Backend VM is in private subnet - not directly accessible from internet.</p>' >> /var/www/html/index.html
```

9. Click **Review + Create** → **Create**

✅ **Expected Result:** Web VM with public IP. Apache serves the sample page. Wait 2–3 minutes for cloud-init.

---

##  Step 8: Deploy Backend VM (Private Subnet)

**Purpose:** VM in private subnet with no public IP—only reachable from web tier or Bastion.

1. Search **Virtual machines** → **+ Create** → **Azure virtual machine**
2. **Basics:**
   * **Resource group:** `rg-capstone09-dev-centralindia`
   * **VM name:** `vm-backend-01`
   * **Region:** Central India
   * **Image:** Ubuntu Server 22.04 LTS
   * **Size:** Standard B2s
3. **Administrator account:**
   * **Authentication type:** SSH public key
   * **Username:** `azureuser`
   * **SSH public key source:** Use existing → `vm-web-key` (or generate new)
   * ⬇️ If new key, download and save
4. **Inbound port rules:** **None** (no public ports)
5. **Next: Disks** → Defaults
6. **Next: Networking:**
   * **Virtual network:** `vnet-capstone09-dev`
   * **Subnet:** `subnet-private`
   * **Public IP:** None
   * **NIC NSG:** None (NSG on subnet)
7. **Next: Management** → Boot diagnostics: Enable
8. **Next: Advanced** → **Custom data**, paste:

```yaml
#cloud-config
package_update: true
packages:
  - apache2
runcmd:
  - systemctl start apache2
  - systemctl enable apache2
  - echo '<h1>Backend API - Private Subnet</h1><p>This VM has NO public IP. Only accessible from web tier or Bastion.</p>' > /var/www/html/index.html
```

9. Click **Review + Create** → **Create**

✅ **Expected Result:** Backend VM with private IP only. Apache runs for internal testing.

---

##  Step 9: Test Sample Web App from Internet

**Purpose:** Confirm the public subnet VM is reachable from the internet.

1. Open **vm-web-01** → **Overview**
2. Copy **Public IP address**
3. In a browser, go to: `http://<public-ip>`
4. Verify the page: "Lab 09 - Public Subnet Web App"

✅ **Expected Result:** Web app loads from the internet. This confirms the public subnet design.

---

##  Step 10: Connect to Backend VM via Bastion

**Purpose:** Verify secure access to the private VM (no public IP).

1. Open **vm-backend-01** → **Connect**
2. Select **Bastion** tab
3. **Username:** `azureuser`
4. **Authentication type:** SSH Private Key
5. Paste the private key (from Step 7)
6. Click **Connect**
7. In the session, run:

```bash
hostname
curl http://localhost
ip addr show
```

✅ **Expected Result:** Bastion session works. Backend VM has only a private IP (e.g., 10.30.2.4). Apache responds on localhost.

---

##  Step 11: Validate Internal Connectivity (Web → Backend)

**Purpose:** Confirm the web tier can reach the backend over the private network.

1. Connect to **vm-web-01** via Bastion (or SSH using its public IP)
2. Get backend private IP: **vm-backend-01** → **Overview** → **Private IP address**
3. From vm-web-01, run:

```bash
curl http://<backend-private-ip>
ping -c 2 <backend-private-ip>
```

✅ **Expected Result:** Web VM reaches backend VM over the internal network. No internet path required.

---

##  Step 12: Verify Private Subnet Isolation

**Purpose:** Confirm the backend is not reachable directly from the internet.

1. From your **local machine** (not Azure), try to reach the backend's private IP—you cannot (it's not routable from internet)
2. The backend has **no public IP**—confirm in **vm-backend-01** → **Overview**
3. Try `http://<backend-private-ip>` from your browser—it will fail (private IP not routable)

✅ **Expected Result:** Backend is only reachable from within the VNet (web VM or Bastion). This demonstrates proper network segmentation.

---

#  PHASE 4: Optional Enhancements (25 Minutes)

---

##  Step 13: Add Simple API Proxy (Optional)

**Purpose:** Demonstrate web tier calling backend and returning data to the user.

1. Get **backend private IP** from Azure Portal: **vm-backend-01** → **Overview** → **Private IP address** (e.g., `10.30.2.4`)
2. Connect to **vm-web-01** via Bastion
3. Create a simple proxy page (replace `10.30.2.4` with your backend IP):

```bash
BACKEND_IP=10.30.2.4
sudo bash -c "echo '<h2>Backend Response (via Web Tier)</h2><pre>' > /var/www/html/backend.html"
curl -s http://$BACKEND_IP | sudo tee -a /var/www/html/backend.html
sudo bash -c "echo '</pre>' >> /var/www/html/backend.html"
```

4. From your browser, open: `http://<vm-web-01-public-ip>/backend.html`

✅ **Expected Result:** Page shows backend content fetched by the web tier—users never connect directly to the backend.

💡 **Learning Point:** Web tier acts as a proxy—users never connect directly to the backend.

---

##  Step 14: Document Architecture and Runbook

**Purpose:** Handover documentation for operations.

**Architecture diagram** should show:
* Public subnet (vm-web-01, public IP)
* Private subnet (vm-backend-01, no public IP)
* Bastion
* NSGs and traffic flow

**Runbook** should include:
* How to access web app (public IP)
* How to access backend (Bastion only)
* How to verify connectivity (web → backend)
* NSG rules and their purpose

---

##  Lab Validation Checklist

✔ Resource group created with tags
✔ VNet with public, private, and Bastion subnets
✔ NSG for public subnet (HTTP, SSH)
✔ NSG for private subnet (internal only)
✔ Bastion deployed
✔ Web VM in public subnet with public IP
✔ Backend VM in private subnet with no public IP
✔ Sample web app accessible from internet
✔ Backend VM accessible via Bastion only
✔ Web VM can reach backend VM internally
✔ Backend not reachable from internet
✔ Architecture diagram and runbook completed

---

##  Key Takeaways

* **Public subnet:** For internet-facing workloads (web tier)
* **Private subnet:** For internal-only workloads (backend, data)
* **NSGs:** Enforce traffic rules per subnet
* **Bastion:** Secure admin access without exposing VMs
* **Segmentation:** Reduces attack surface; backend stays isolated
* **DMZ pattern:** Internet → public tier → private tier
