## Module 02 — v2v-helper

### Overview

- v2v-helper is the heart of vjailbreak.  
- It performs the actual VM migration from VMware to OpenStack (worker program).  
- It runs as a containerized Kubernetes Job and handles disk copying, format conversion, and target VM creation.

---

## Components Inside v2v-helper Pod

1. **v2v-helper Binary (Main Program)**  
   - Reads migration configs  
   - Coordinates all tools  
   - Reports status back to Kubernetes  

2. **VMware VDDK**  
   - VMware’s official disk access library  
   - Reads VMware disks via snapshots  
   - Supports CBT (Changed Block Tracking)  

3. **NBD Server + Client Utilities**  
   - Exposes VMware disks as block devices  

4. **virt-v2v**  
   - Open-source conversion tool  
   - Converts VMDK → QCOW2 (KVM format)  
   - Removes VMware Tools  
   - Installs VirtIO drivers  
   - Ensures VM boots correctly on OpenStack  

5. **OpenStack Client Libraries**  
   - SDKs / CLIs to interact with OpenStack APIs  

6. **Linux Block & Filesystem Tools**  
   - Disk handling and filesystem operations  

7. **Kubernetes Client / Event Reporter**  
   - Reports migration progress & events  

8. **Configurations & Secrets (Mounted)**  
   - Credentials and mapping configs  

---

## Migration Flow / Execution Steps

### 1. v2v-helper Pod Creation

- User creates a `MigrationPlan`  
- Controller processes the plan  
- For each VM → one v2v-helper pod is created  

---

### 2. Initialization

v2v-helper starts and reads configuration:

- VM name / ID  
- Migration type (Hot / Cold)  
- Network mapping  
- Storage mapping  

---

### 3. Connect to VMware vCenter

Using `VMwareCreds`:

- Connects via vSphere APIs  
- Fetches:
  - VM metadata  
  - Disk list  
  - Disk sizes  
  - Power state  

This helps v2v-helper understand the VM layout.

---

### 4. Connect to OpenStack

Using `OpenstackCreds`:

- Authenticates via Keystone  
- Verifies access to:
  - Cinder  
  - Nova  
  - Neutron  

---

### 5. Create OpenStack Volumes

Steps:

- Reads disk size from VMware  
- Uses `StorageMapping`  
- Creates matching Cinder volumes  
- Marks boot disk as bootable  

**Example**
VMware Disk 1 (50 GB) → OpenStack Volume 1 (50 GB)
VMware Disk 2 (100 GB) → OpenStack Volume 2 (100 GB)


Volumes are empty initially.

---

### 6. Attach Volumes to v2v-helper Pod

After creation:

- Volumes attach to the pod  

Example device mapping:

/dev/vdb
/dev/vdc


---

### 7. Disk Copy Begins (NBD — Critical Step)

NBD streams disk data block-by-block.

Mapping example:

/dev/nbd0 → VMware disk
/dev/vdb → OpenStack volume


Copy logic:
Read block from /dev/nbd0
Write block to /dev/vdb
Repeat until complete


---

### 8. Snapshot Handling

Depends on migration type:

#### Cold Migration

- VM powered off  
- Disk static  
- Single full copy  

#### Hot Migration

- Snapshot created  
- Initial full copy  
- Incremental copies via CBT  
- Final cutover sync  

---

### 9. OS Conversion (virt-v2v)

After disk copy:

virt-v2v performs:

- Removes VMware Tools  
- Installs VirtIO drivers  
- Fixes bootloader  
- Adjusts NIC configuration  

---

### 10. Create VM in OpenStack

v2v-helper calls Nova:

- Uses created volumes  
- Attaches networks via `NetworkMapping`  
- Assigns flavor  
- Creates target VM  

---

### 11. Boot & Verification

- Boots the VM  
- Runs health checks  
- Executes first-boot scripts  
- Confirms VM readiness  

---

### 12. Status Reporting & Exit

- Updates `MigrationPlan.status`  
- Emits Kubernetes events  
- Pod exits after completion  

---

## Pod Lifecycle

Created → Running → Completed → Terminated

[Module 03 — vpwned](module-03-vpwned.md)
