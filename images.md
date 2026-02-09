<img width="2248" height="1464" alt="vjb" src="https://github.com/user-attachments/assets/d95bd7da-f5e8-46e9-bc6d-e935f045a6c4" />

---

1. User fills form in UI
   ↓
2. UI creates MigrationPlan via Kubernetes API
   ↓
3. MigrationPlanReconciler sees new plan
   ↓
4. Creates Migration objects (one per VM)
   ↓
5. MigrationReconciler sees Migration object
   ↓
6. Creates v2v-helper Pod
   ↓
7. Pod starts, runs v2v-helper/main.go
   ↓
8. v2v-helper connects to VMware & OpenStack
   ↓
9. Creates volumes, snapshots VM
   ↓
NBD server streams disk data
    ↓
11. Data copied to OpenStack volumes
    ↓
12. For hot migrations: Repeat changed blocks copy
    ↓
13. Admin clicks cutover OR auto-cutover triggers
    ↓
14. Final sync, power off source, convert disk
    ↓
15. Create target VM in OpenStack
    ↓
16. MigrationReconciler sees pod completed
    ↓
18. Updates Migration status to "Succeeded"
    ↓
19. UI polls, shows success ✓

---
