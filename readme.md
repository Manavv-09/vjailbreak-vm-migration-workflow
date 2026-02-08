
# vjailbreak VM Migration Workflow  
**Case Study: VMware ESXi → OpenStack**

---

## 1. About the Tool

vjailbreak is an open-source migration tool developed by Platform9.  
It helps organizations migrate virtual machines from VMware environments to OpenStack-compatible clouds or PCD platforms with reduced complexity and downtime.

It is often referred to as a “VMware escape tool”.

---

## 2. Objective

This case study documents my learning and hands-on experience performing a virtual machine migration using the vjailbreak tool.

The objective was to understand the tool’s architecture and observe real-world infrastructure transition workflows, including:

- Appliance bootstrapping  
- Disk migration  
- Network and storage mapping  
- Post-migration validation  

This migration involved moving workloads from **VMware ESXi** to an **OpenStack** environment.

Rather than relying solely on documentation, I focused on analyzing internal workflows, identifying failure points, and troubleshooting issues encountered during the migration process.

---

## 3. High-Level Migration Overview

At a high level, the migration workflow operates as follows:

- Connects to the source VMware vCenter / ESXi environment to discover virtual machines  
- Allows selection of VMs to migrate  
- Converts VMware disk formats (VMDK) into OpenStack-compatible formats such as QCOW2 or RAW  
- Recreates the VMs in OpenStack / PCD with mapped CPU, RAM, network, and storage configurations  
- Automates the end-to-end migration workflow  

---

## 4. vjailbreak Migration Architecture (Code-Level Understanding)

### Module 01 — Project Orientation (vjb VM)

vjailbreak is designed as a self-contained migration appliance delivered as a QCOW2 image.

It enables migration of VMs from VMware vCenter to OpenStack (KVM-based environments).

The vjailbreak VM runs a **k3s Kubernetes cluster** which hosts all migration components.

---

### Core Components

#### K3s Kubernetes Cluster

The foundation of vjailbreak is a lightweight K3s Kubernetes distribution running entirely inside the appliance VM.

Key components include:

- **K3s Server** — Kubernetes control plane node  
- **containerd** — Container runtime  
- **etcd** — Kubernetes state database  
- **kubeconfig** — Cluster authentication configuration  

---

#### Application Components

**vjailbreak UI**  
A web interface for managing migrations, deployed as a Kubernetes Deployment.

**Migration Controller**  
The orchestration engine that:

- Manages custom resources  
- Handles worker node provisioning  
- Coordinates migration execution  

**v2v-helper Pods**

Responsible for VM conversion using `virt-v2v`:

- Convert VMDK → QCOW2  
- Inject VirtIO drivers (Windows)  
- Handle disk transfer & transformation  

---

### Custom Resources (CRDs)

vjailbreak defines several Kubernetes CRDs:

- **OpenstackCreds** — OpenStack authentication details  
- **VMwareCreds** — VMware / vCenter credentials  
- **VMwareMachines** — Discovered VM inventory  
- **NetworkMapping** — VMware → OpenStack network mapping  
- **StorageMapping** — Datastore → Cinder mapping  
- **VjailbreakNode** — Migration infra nodes  
- **MigrationTemplate** — Reusable configs  
- **MigrationPlan** — Workflow definitions  
- **Migration** — Execution resource  

---

### Initialization Process

#### First Boot Sequence

On first appliance boot:

1. Cron job triggers `/etc/pf9/install.sh`  
2. K3s single-node cluster initializes  
3. Preloaded container images are imported  
4. Kubernetes manifests are applied  
5. Migration services start  

---

### Critical Directories & Files

#### Configuration Paths

- `/etc/pf9/` — Core configs & scripts  
- `k3s.env` — Cluster config  
- `k3s/token` — Join token  
- `images/` — Container image bundle  
- `htpasswd` — UI authentication  

#### Migration Paths

- `/home/ubuntu/` — Workspace  
- `vmware-vix-disklib-distrib/` — VDDK libraries  
- `virtio-win/` — Windows drivers  
- `/var/log/pf9/` — Migration logs  

---

### Network Configuration

- Primary interface via DHCP or static IP  
- DNS resolution required for VMware connectivity  

#### Required Connectivity

**VMware**

- vCenter — 443  
- ESXi hosts — 902, 22  

**OpenStack APIs**

- Keystone  
- Nova  
- Cinder  
- Neutron  

**Internet**

- Required for driver downloads  

---

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



				




