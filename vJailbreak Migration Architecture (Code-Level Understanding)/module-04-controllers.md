## Module 04 — Kubernetes Controllers & Migration Orchestration

---

### Core Idea

The vJailbreak system uses Kubernetes Controllers to orchestrate the entire migration workflow.

These controllers watch **Custom Resources (CRs)** and continuously reconcile their **actual state** toward the **desired state**, managing everything from credential validation to full VM migration execution.

---

## Migration Orchestration Flow

### 1. vJailbreak Appliance VM Deployment

A single vJailbreak appliance VM is deployed.

This VM already contains:

- Kubernetes cluster  
- vJailbreak components (UI, controllers, services)  

Once deployed:

- Kubernetes is up  
- Controller Manager is running  
- UI is accessible  

---

### 2. User Accesses the UI

The user opens the vJailbreak UI via browser to begin migration setup.

---

### 3. User Provides Environment Configuration

Through the UI, the user enters:

- VMware vCenter details  
- OpenStack credentials  
- Network mapping  
- Storage mapping  

---

#### UI Action

The UI communicates with Kubernetes API and creates CRDs:

- `VMwareCreds`  
- `OpenstackCreds`  
- `NetworkMapping`  
- `StorageMapping`  
- `MigrationTemplate`  

---

### 4. Controllers React to CRDs

Controllers continuously watch these resources.

When CRDs are created or updated:

- Credentials are validated  
- Infrastructure inventory is discovered  
- CRD status fields are updated  

---

### 5. User Creates a MigrationPlan

From the UI:

- User selects a VM  
- Clicks **Start Migration**  

This action creates a:

MigrationPlan CRD


---

### 6. Controller Reconciliation Logic

The `MigrationPlan` reconciler detects the new resource.

It enters the Kubernetes controller loop: Reconcile()


Here the controller evaluates:

- Migration readiness  
- Resource mappings  
- Credential validation  

---

### 7. Controller Triggers Migration Execution

If validation succeeds:

- Controller creates a `v2v-helper` Job/Pod  
- Migration execution begins  

Controller updates status: MigrationPlan.status.phase = Running


---

### 8. Controller Tracks Migration Progress

The controller continuously monitors the pod state.

#### Success Path

status.phase = Completed


#### Failure Path

status.phase = Failed


Progress updates are reflected in:

- Kubernetes events  
- MigrationPlan status  
- UI dashboards  

---
