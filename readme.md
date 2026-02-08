
# vjailbreak VM Migration Workflow  
**Case Study: VMware ESXi → OpenStack**

---

## 1. About the Tool

vjailbreak is an open-source migration tool developed by Platform9.  
It helps organizations migrate virtual machines from VMware environments to OpenStack-compatible clouds or PCD platforms with reduced complexity and downtime.

It is often referred to as a “VMware escape tool” due to its role in enabling platform transitions.

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

				




