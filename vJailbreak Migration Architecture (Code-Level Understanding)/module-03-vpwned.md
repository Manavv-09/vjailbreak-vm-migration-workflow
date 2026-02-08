## Module 03 — vpwned Agent

### Overview

- The vpwned agent runs as a Kubernetes pod in the `migration-system` namespace.  
- It is deployed alongside the Migration Controller.  
- Acts as an API service that the Migration Controller and UI call to perform VMware vCenter operations.  
- Functions as a centralized control-plane service.

---

## Core Components

### 1. gRPC Service Implementation

The vpwned agent is built using **gRPC**, with **gRPC-Gateway** enabling REST API exposure.

This allows both:

- Internal gRPC communication  
- External REST API calls  

The service definitions handle infrastructure validation and operational requests.

---

### 2. API Endpoints

The service exposes key endpoints via **HTTP POST**:

---

#### 2.1 ValidateOpenstackIp

Validates OpenStack IP availability.

Functions:

- Checks whether provided IPs are already in use  
- Detects duplicate IP entries  
- Returns validation status for each IP  
- Provides failure reasons if conflicts exist  

---

#### 2.2 RevalidateCredentials

Re-validates infrastructure credentials.

Functions:

- Validates VMware credentials  
- Validates OpenStack credentials  
- Updates credential status in Kubernetes  
- Triggers resource discovery after successful validation  

---

#### 2.3 InjectEnvVariables

Injects environment variables into running pods.

Use cases:

- Dynamic configuration updates  
- Runtime parameter injection  
- Migration environment adjustments  

---

## Key Functions

The vpwned agent provides several operational capabilities:

### Version Management

- Handles upgrade orchestration  
- Tracks upgrade progress  
- Performs cleanup after version transitions  

---

### Cleanup Operations

- Executes pre-upgrade cleanup  
- Reclaims post-migration resources  
- Removes temporary migration artifacts  

---

### Validation Services

- Credential validation  
- Infrastructure access verification  
- IP availability checks  

---

### Bare Metal Management

- Lifecycle management of bare-metal hosts  
- Provisioning coordination  
- Resource tracking within migration workflows  

---

[Module 04 — Controllers](module-04-controllers.md)
