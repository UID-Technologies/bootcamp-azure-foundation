#  Lab 2: Azure Virtual Network (VNet) & Network Security Groups (NSGs)

##  Lab Objectives

By the end of this lab, participants will be able to:

* Design Azure networking using **VNets and subnets**
* Apply **CIDR-based IP address planning**
* Secure traffic using **Network Security Groups (NSGs)**
* Understand **inbound vs outbound rules**
* Access VMs securely using **Azure Bastion**
* Validate network security and connectivity

All tasks are performed in Microsoft Azure.

---

##  Estimated Time

**~120 minutes**

---

## Prerequisites

* Completed **Session 1 – Azure Fundamentals & Governance lab**
* Active Azure subscription
* Owner / Contributor role on subscription
* Azure Portal access

---

##  Lab Scenario (Enterprise Context)

You are designing a **secure application network** with:

* Separate subnets for application and management
* Controlled inbound access
* No public IP exposure to application VMs
* Secure administrative access using Azure Bastion

![Image](https://miro.medium.com/1%2ARzgpIFBZVpnHytRF07OufQ.png)

![Image](https://learn.microsoft.com/en-us/azure/virtual-network/media/network-security-group-how-it-works/network-security-group-interaction.png)

![Image](https://miro.medium.com/1%2A8dKBDQl0-WP_oDSRhCWW9w.png)

![Image](https://learn.microsoft.com/en-us/azure/virtual-network/media/manage-network-security-group/view-security-rules.png)

![Image](https://learn.microsoft.com/en-us/azure/virtual-network/media/manage-network-security-group/add-security-rule.png)

---

##  Step 1: Review Azure Networking Concepts (Checkpoint)

Before implementation, confirm understanding:

* VNet = private network in Azure
* Subnets = segmentation inside VNet
* NSG = virtual firewall
* Azure Bastion = secure RDP/SSH without public IP

Trainer checkpoint ✔️

---

##  Step 2: Create a Virtual Network (VNet)

1. In **Azure Portal**, search **Virtual Networks**
2. Click **Create**
3. Configure **Basics**:

   * **Subscription:** Active subscription
   * **Resource Group:** `rg-app-dev-centralindia`
   * **Name:** `vnet-app-dev`
   * **Region:** Central India
4. Click **Next: IP Addresses**

---

##  Step 3: Configure Address Space & Subnets

### Address Space

* IPv4 address space:

  ```
  10.10.0.0/16
  ```

### Subnet 1 – Application Subnet

* **Name:** `subnet-app`
* **Address range:** `10.10.1.0/24`

### Subnet 2 – Management Subnet

* **Name:** `subnet-mgmt`
* **Address range:** `10.10.2.0/24`

Click **Review + Create** → **Create**

💡 **CIDR Best Practice:**
Leave space for future subnets (databases, gateways, etc.)

---

##  Step 4: Create Network Security Group (NSG)

1. Search **Network Security Groups**
2. Click **Create**
3. Configure:

   * **Name:** `nsg-app-subnet`
   * **Resource Group:** `rg-app-dev-centralindia`
   * **Region:** Central India
4. Click **Review + Create** → **Create**

---

##  Step 5: Understand Default NSG Rules (Read-Only)

1. Open `nsg-app-subnet`
2. Review **Inbound security rules**
3. Observe default rules:

   * Allow VNet inbound
   * Allow Azure Load Balancer
   * Deny all inbound

💡 **Learning Point:**
Azure follows **default deny** for security.

---

##  Step 6: Create Inbound NSG Rule (HTTP)

1. Go to **Inbound security rules**
2. Click **Add**
3. Configure:

   * **Source:** Any
   * **Source port ranges:** *
   * **Destination:** Any
   * **Destination port:** 80
   * **Protocol:** TCP
   * **Action:** Allow
   * **Priority:** 100
   * **Name:** `allow-http`
4. Click **Add**

---

##  Step 7: Create Inbound NSG Rule (SSH – Restricted)

1. Click **Add**
2. Configure:

   * **Source:** IP Addresses
   * **Source IP:** *Your public IP*/32
   * **Destination port:** 22
   * **Protocol:** TCP
   * **Action:** Allow
   * **Priority:** 110
   * **Name:** `allow-ssh-admin`
3. Click **Add**

⚠️ **Security Note:**
Never use `0.0.0.0/0` in production.

---

##  Step 8: Associate NSG with Subnet

1. Open `nsg-app-subnet`
2. Select **Subnets**
3. Click **Associate**
4. Select:

   * **VNet:** `vnet-app-dev`
   * **Subnet:** `subnet-app`
5. Click **OK**

✅ NSG now protects **all resources** in subnet-app.

---

##  Step 9: Deploy Test VM into Subnet

1. Go to **Virtual Machines → Create**
2. Configure:

   * **Name:** `vm-app-01`
   * **Resource Group:** `rg-app-dev-centralindia`
   * **Region:** Central India
   * **Image:** Ubuntu LTS
   * **Size:** Standard B2s
3. Networking:

   * **VNet:** `vnet-app-dev`
   * **Subnet:** `subnet-app`
   * **Public IP:** None
4. Click **Review + Create** → **Create**

---

##  Step 10: Deploy Azure Bastion (Secure Access)

1. Search **Bastions**
2. Click **Create**
3. Configure:

   * **Name:** `bastion-app`
   * **Resource Group:** `rg-app-dev-centralindia`
   * **VNet:** `vnet-app-dev`
   * **Subnet:** Create `AzureBastionSubnet`

     ```
     10.10.10.0/27
     ```
4. Click **Create**

⏳ Bastion deployment may take ~10 minutes.

---

##  Step 11: Connect to VM via Bastion

1. Open **vm-app-01**
2. Click **Connect → Bastion**
3. Enter VM credentials
4. Connect via browser

✅ **Expected Result:**
Secure SSH session without public IP

---

##  Step 12: Validate NSG Enforcement

Inside VM:

```bash
curl http://localhost
```

From external machine:

* HTTP works
* SSH blocked unless via Bastion

💡 **Learning Point:**
NSGs enforce traffic **before VM OS firewall**

---

##  Step 13: (Optional) VNet Peering Overview

Trainer explains:

* When to use VNet peering
* Hub-and-spoke model
* No gateway required
* Transitive routing concepts

(No configuration required for this lab)

---

##  Step 14: Validate Networking via Azure CLI (Optional)

In **Cloud Shell**:

```bash
az network vnet list
az network nsg list
az network vnet subnet list --vnet-name vnet-app-dev -g rg-app-dev-centralindia
```

---

##  Lab Validation Checklist

✔ VNet created
✔ Subnets configured correctly
✔ NSG created and associated
✔ Secure inbound rules applied
✔ VM deployed without public IP
✔ Bastion used for access
✔ Network security validated

---

##  Key Takeaways

* VNets are the foundation of Azure networking
* CIDR planning prevents future redesigns
* NSGs are stateful, subnet/VM-level firewalls
* Default deny improves security posture
* Azure Bastion removes need for public IPs
* Secure networking is **design-first**
