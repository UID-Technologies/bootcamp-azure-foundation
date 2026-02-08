#  Lab 4: Azure Storage – Step-by-Step

##  Lab Objectives

By the end of this lab, participants will be able to:

* Create and configure **Azure Storage Accounts**
* Understand **redundancy options** (LRS, ZRS, GRS)
* Work with **Blob Storage** (containers & blobs)
* Apply **lifecycle management policies**
* Secure access using **Shared Access Signatures (SAS)**
* Create and mount **Azure File Shares** on a Linux VM
* Validate storage access and governance

All tasks are performed in Microsoft Azure.

---

##  Estimated Time

**~120 minutes**

---

## Prerequisites

* Completed **Sessions 1–3 labs**
* Existing Linux VM:

  * `vm-app-linux-01`
* Resource Group:

  * `rg-app-dev-centralindia`
* Azure Bastion access to VM
* Contributor access on subscription

---

##  Lab Scenario (Enterprise Context)

You are enabling **application storage** for a VM-based workload:

* Store application artifacts and logs
* Control access securely
* Optimize long-term storage costs
* Support shared file access across VMs

![Image](https://learn.microsoft.com/en-us/security/zero-trust/media/secure-storage/azure-infra-storage-network-2.svg)

![Image](https://learn.microsoft.com/en-us/azure/storage/common/media/storage-redundancy/geo-redundant-storage.png)

![Image](https://www.nalashaa.com/images/blog-inner-images/azure-storage-blob.png)

![Image](https://cloud.telestream.net/images/tutorials/azure-structure-679c37b5.png)

![Image](https://learn.microsoft.com/en-us/azure/storage/blobs/media/blob-containers-portal/menu-expand-lrg.png)

---

##  Step 1: Review Azure Storage Concepts (Checkpoint)

Confirm understanding:

* Storage Account = security + billing boundary
* Blob Storage = object storage
* Azure Files = managed file shares
* Redundancy impacts **durability, availability, and cost**

Trainer checkpoint ✔️

---

##  Step 2: Create a Storage Account

1. In **Azure Portal**, search **Storage accounts**
2. Click **Create**

### Basics

* **Subscription:** Active subscription
* **Resource Group:** `rg-app-dev-centralindia`
* **Storage account name:**
  `stappdev<uniqueid>` (must be globally unique)
* **Region:** Central India
* **Performance:** Standard
* **Redundancy:** LRS

Click **Next: Advanced**

---

##  Step 3: Configure Security & Access Settings

1. Disable **public access** for blobs
2. Leave:

   * Secure transfer required = Enabled
   * Minimum TLS version = 1.2
3. Click **Review + Create → Create**

✅ **Expected Result:**
Secure GPv2 storage account created

---

##  Step 4: Review Redundancy Options (Conceptual)

Open **Data management → Redundancy**

Discuss:

* **LRS** – Single datacenter (lowest cost)
* **ZRS** – Zone resilient
* **GRS** – Cross-region replication
* **GZRS** – Zone + region resilience

💡 **Best Practice:**
Choose redundancy based on **business criticality**, not defaults.

---

##  Step 5: Create Blob Container

1. Open storage account → **Containers**
2. Click **+ Container**
3. Configure:

   * **Name:** `app-data`
   * **Public access level:** Private
4. Click **Create**

---

##  Step 6: Upload and Manage Blobs

1. Open `app-data` container
2. Click **Upload**
3. Upload any local file (e.g., text or image)
4. Verify blob appears

💡 **Learning Point:**
Blob storage is **flat**, folders are virtual.

---

##  Step 7: Configure Lifecycle Management Policy

1. Go to **Data management → Lifecycle management**
2. Click **Add a rule**

### Rule 1 – Cool Storage

* Name: `move-to-cool`
* Apply to: Base blobs
* Condition: Last modified > 30 days
* Action: Move to Cool tier

### Rule 2 – Delete Old Data

* Name: `delete-old-blobs`
* Condition: Last modified > 365 days
* Action: Delete blob

Click **Add**

✅ Automates **cost optimization**

---

##  Step 8: Generate SAS Token for Blob Access

1. Open **Containers → app-data**
2. Select any blob
3. Click **Generate SAS**
4. Configure:

   * Permissions: Read
   * Expiry: +1 day
5. Generate and copy **Blob SAS URL**

💡 **Security Rule:**
Use SAS instead of sharing account keys.

---

##  Step 9: Test SAS Access

1. Open new browser window
2. Paste SAS URL
3. Confirm:

   * Blob downloads successfully
   * Access works without Azure login

---

##  Step 10: Create Azure File Share

1. Go to storage account → **File shares**
2. Click **+ File share**
3. Configure:

   * **Name:** `appfiles`
   * **Quota:** 100 GB
4. Click **Create**

---

##  Step 11: Mount Azure File Share on Linux VM

1. Open file share → **Connect**
2. Select:

   * **Linux**
   * **Authentication:** Storage account key
3. Copy mount commands

SSH into VM via **Azure Bastion** and run:

```bash
sudo apt-get update
sudo apt-get install -y cifs-utils
sudo mkdir /mnt/appfiles
sudo mount -t cifs //<storageaccount>.file.core.windows.net/appfiles /mnt/appfiles \
-o vers=3.0,username=<storageaccount>,password=<accountkey>,dir_mode=0777,file_mode=0777
```

---

##  Step 12: Validate File Share Access

Inside VM:

```bash
cd /mnt/appfiles
echo "Hello from Azure Files" > test.txt
ls -l
```

Check in Azure Portal → File share → file visible

✅ VM ↔ Azure Files integration successful

---

##  Step 13: Storage Monitoring (Intro)

1. Open storage account → **Insights**
2. Review:

   * Capacity
   * Transactions
   * Latency

💡 Monitoring helps detect abnormal usage early.

---

##  Step 14: Governance & Security Review

Checklist:

* Public access disabled
* Lifecycle policy enabled
* SAS used instead of account key sharing
* Storage located in same region as VM
* Redundancy chosen intentionally

---

##  Lab Validation Checklist

✔ Storage account created
✔ Secure access settings applied
✔ Blob container created
✔ Blobs uploaded successfully
✔ Lifecycle rules configured
✔ SAS access validated
✔ Azure File Share created
✔ File share mounted on VM

---

##  Key Takeaways

* Storage Accounts are security & billing boundaries
* Redundancy impacts cost and availability
* Blob storage is ideal for unstructured data
* Lifecycle policies automate cost control
* SAS provides controlled, temporary access
* Azure Files enables shared storage for VMs
