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
17. Updates Migration status to "Succeeded"
    ↓
18. UI polls, shows success ✓
