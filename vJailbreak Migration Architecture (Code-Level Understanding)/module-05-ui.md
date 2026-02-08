## Module 05 — vJailbreak UI Architecture

---

### Overview

The vJailbreak UI is a React-based web application that provides a user-friendly portal for managing VM migrations.

It runs as a containerized service within the Kubernetes cluster and exposes a responsive interface for:

- Configuring credentials  
- Creating migration plans  
- Monitoring migration progress  
- Managing infrastructure resources  


---

## Deployment Architecture

### Kubernetes Deployment

- The UI is deployed as a **single-replica Deployment**  
- Namespace: `migration-system`  

---

### Network Exposure

The UI is exposed via **nginx-ingress** with SSL termination.

**Configuration Details:**

- **Service:** ClusterIP on port 80  
- **Ingress:** Routes `/` path to UI service  
- **SSL:** Automatic HTTP → HTTPS redirect  
- **API Routes:** Separate ingress for Kubernetes API endpoints  

---

## Authentication

Authentication is handled via **Basic Auth** using `htpasswd`.

**Implementation Details:**

- Password file mounted at:  
/etc/nginx/shadow


- Managed via `vjbctl` CLI tool  
- User management handled using:  
pf9-htpasswd.sh

---

- Users starts migration through drawer-based forms in the vJailbreak UI.
- Which creates K8s resources via REST APIs
- Progress is tracked through a polling mechanism on the migration page that displays real time updates.
  
---

## Key Features

### 1. Migration Management

#### Migration Form Drawer

Slide-out panel used to create new migrations.

**Capabilities:**

- Source VM selection  
- Destination configuration  
- Network mapping  
- Storage mapping  
- Migration options:
- Hot / Cold migration  
- Cutover type  

---

#### Migrations Table

Provides a comprehensive view of all migrations.

**Features:**

- Real-time status updates  
- Progress tracking  
- Action buttons:
- Retry  
- Delete  
- View logs  
- Bulk operations support  

---

### 2. Credential Management

#### Credentials Page

Used to manage VMware and OpenStack credentials.

**Capabilities:**

- Add / Edit / Delete credential sets  
- Validation status indicators  
- Test connection functionality  
- Credential usage tracking  

---

### 3. Onboarding Experience

#### Onboarding Page

Designed for first-time users.

**Features:**

- Welcome message and branding  
- Quick-start migration button  
- Documentation links  
- Guided product tour integration  

---

## User Experience Features

### Real-Time Updates

The UI uses **React Query** for data synchronization.

**Capabilities:**

- Auto-refresh based on migration status  
- Optimistic UI updates  
- Background refetching with configurable intervals  

---

### Progress Monitoring

#### Migration Progress Component

Provides visual tracking of migration status.

**Features:**

- Phase-based progress bars  
- Detailed status messages  
- Time elapsed calculations  
- Error state handling  

---

### Responsive Design

Built using **Material-UI’s responsive design system**.

**Capabilities:**

- Mobile-friendly layouts  
- Adaptive data tables  
- Progressive disclosure of information  

---


