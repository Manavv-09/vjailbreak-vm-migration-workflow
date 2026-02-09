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
7. Pod starts, runs v2v-helper - v2v-helper connects to VMware & OpenStack
   ↓
8. Creates volumes, snapshots VM
   ↓
9. NBD server streams disk data
    ↓
10. Data copied to OpenStack volumes
    ↓
11. For hot migrations: Repeat changed blocks copy
    ↓
12. Admin clicks cutover OR auto-cutover triggers
    ↓
13. Final sync, power off source, convert disk
    ↓
14. Create target VM in OpenStack
    ↓
15. MigrationReconciler sees pod completed
    ↓
16. Updates Migration status to "Succeeded"
    ↓
17. UI polls, shows success ✓

---
