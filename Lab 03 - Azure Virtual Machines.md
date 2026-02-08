#  Lab 3: Azure Virtual Machines – Step-by-Step

## Lab Objectives

By the end of this lab, participants will be able to:

* Provision **Linux / Windows Azure VMs**
* Select appropriate **VM sizes and images**
* Understand **OS disk vs data disks**
* Configure **availability options** (sets & zones)
* Use **cloud-init** for VM bootstrapping
* Enable **boot diagnostics**
* Securely access VMs via **Azure Bastion**
* Validate VM networking and storage

All tasks are performed in Microsoft Azure.

---

##  Estimated Time

**~120 minutes**

---

##  Prerequisites

* Completed **Session 1 & 2 labs**
* Existing resources:

  * Resource Group: `rg-app-dev-centralindia`
  * VNet: `vnet-app-dev`
  * Subnet: `subnet-app`
  * NSG attached to subnet
  * Azure Bastion deployed
* Contributor access on subscription

---

##  Lab Scenario (Enterprise Context)

You are deploying a **compute layer** for an application that requires:

* Secure network placement
* Automated OS configuration
* Additional storage for application data
* Diagnostic visibility for troubleshooting
* High availability awareness

![Image](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/n-tier/images/single-vm-diagram.svg)

![Image](https://learn.microsoft.com/en-us/azure/virtual-machines/media/disks-high-availability/disks-availability-set.png)

![Image](https://learn.microsoft.com/en-us/azure/virtual-machines/media/disks-types/managed-disk-decision-tree.png)

![Image](https://learn.microsoft.com/en-us/azure/virtual-machines/media/disks-performance/example-vm-allocation.png)

![Image](https://learn.microsoft.com/en-us/azure/virtual-machines/media/disks-performance/vm-level-throttling.jpg)


---

##  Step 1: Review VM Architecture (Checkpoint)

Before building, confirm understanding:

* VM = compute + disk + network
* OS Disk vs Data Disk
* NIC attached to subnet
* NSG applied at subnet/VM
* Bastion for secure access

Trainer checkpoint ✔️

---

##  Step 2: Create a Linux VM (Ubuntu)

1. In **Azure Portal**, go to **Virtual Machines**
2. Click **Create → Azure Virtual Machine**

### Basics

* **Subscription:** Active subscription
* **Resource Group:** `rg-app-dev-centralindia`
* **VM Name:** `vm-app-linux-01`
* **Region:** Central India
* **Availability options:** No infrastructure redundancy required (for lab)
* **Image:** Ubuntu Server 22.04 LTS
* **Size:** Standard B2s

Click **Next: Administrator Account**

---

##  Step 3: Configure Administrator & Authentication

* **Authentication type:** SSH public key
* **Username:** `azureuser`
* **SSH key source:** Generate new key pair
* **Key pair name:** `vm-app-linux-key`

Click **Next: Disks**

💡 **Best Practice:**
Always use **SSH keys**, never passwords.

---

##  Step 4: Configure Disks (OS Disk)

* **OS disk type:** Premium SSD (recommended)
* **Encryption:** Platform-managed key
* Leave defaults for lab

Click **Next: Networking**

---

##  Step 5: Configure Networking (Secure by Design)

* **Virtual network:** `vnet-app-dev`
* **Subnet:** `subnet-app`
* **Public IP:** None
* **NIC network security group:** None (NSG already on subnet)
* **Load balancing:** No

Click **Next: Management**

---

##  Step 6: Enable Boot Diagnostics

* **Boot diagnostics:** Enable
* **Storage account:** Managed (default)

💡 **Why this matters:**
Boot diagnostics help troubleshoot **boot failures**.

Click **Next: Advanced**

---

##  Step 7: Add cloud-init Script (Automation)

Paste the following **cloud-init** script:

```yaml
#cloud-config
package_update: true
packages:
  - apache2
runcmd:
  - systemctl start apache2
  - echo "<h1>Welcome to Azure VM</h1>" > /var/www/html/index.html
```

💡 Automates software installation at first boot.

Click **Review + Create → Create**

⬇️ Download SSH key and keep it safe.

---

##  Step 8: Verify VM Deployment

1. Wait for deployment to complete
2. Open **vm-app-linux-01**
3. Note:

   * Private IP address
   * Network interface
   * Boot diagnostics status

---

##  Step 9: Connect to VM Using Azure Bastion

1. Open VM → **Connect**
2. Select **Bastion**
3. Username: `azureuser`
4. Authentication: SSH private key
5. Connect

✅ **Expected Result:**
Browser-based SSH session opens

---

## Step 10: Validate cloud-init Execution

Inside VM, run:

```bash
systemctl status apache2
curl http://localhost
```

✅ Apache is running and serving HTML page

---

##  Step 11: Attach a Data Disk

1. Open VM → **Disks**
2. Click **Create and attach a new disk**
3. Configure:

   * **Name:** `vm-app-linux-data01`
   * **Size:** 32 GiB
   * **Type:** Standard SSD
4. Save

---

##  Step 12: Format and Mount Data Disk

Inside VM:

```bash
lsblk
sudo mkfs.ext4 /dev/sdc
sudo mkdir /mnt/data
sudo mount /dev/sdc /mnt/data
df -h
```

✅ Data disk mounted successfully

💡 **Learning Point:**
OS disk ≠ data disk (separation of concerns)

---

##  Step 13: Availability Options (Conceptual)

Trainer explains:

* Availability Sets (fault & update domains)
* Availability Zones (AZ-level isolation)
* When to use each

(No redeployment required in this lab)

---

##  Step 14: Review VM Extensions & Diagnostics

1. Open VM → **Extensions + Applications**
2. Observe:

   * Linux Agent
3. Open **Boot diagnostics**
4. Review:

   * Screenshot
   * Serial logs

---

##  Step 15: Validate via Azure CLI (Optional)

In **Cloud Shell**:

```bash
az vm list -g rg-app-dev-centralindia -o table
az vm show -n vm-app-linux-01 -g rg-app-dev-centralindia
az disk list -g rg-app-dev-centralindia -o table
```

---

##  Lab Validation Checklist

✔ Linux VM deployed securely
✔ SSH key-based authentication used
✔ cloud-init executed successfully
✔ Apache installed automatically
✔ Data disk attached and mounted
✔ Boot diagnostics enabled
✔ Bastion access validated

---

##  Key Takeaways

* Azure VMs are highly configurable
* cloud-init enables **immutable infrastructure**
* Data disks should be separate from OS
* Bastion removes public IP exposure
* Diagnostics are critical for operations
* Availability design matters in production

---
