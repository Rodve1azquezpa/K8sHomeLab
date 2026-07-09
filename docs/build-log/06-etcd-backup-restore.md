# Phase 6: ETCD Disaster Recovery

## 🏗️ Deployment Overview
The cluster's central state database (ETCD) was successfully backed up and restored to simulate a disaster recovery scenario. The restore process utilized a non-destructive path modification to point the static pod to the new database directory.

| Component | Status | Details |
| :--- | :--- | :--- |
| **ETCD Client** | 🟢 Active | `etcd-client` installed on the control plane. |
| **Backup File** | 🟢 Secured | Snapshot verified and stored at `/tmp/etcd-backup.db`. |
| **Restore Directory** | 🟢 Active | Database successfully extracted to `/var/lib/etcd-backup`. |
| **Static Pod Config** | 🟢 Updated | `/etc/kubernetes/manifests/etcd.yaml` volume mount redirected to the new directory. |

---

## ✅ Task Completion Checklist

### 1. Snapshot Creation
* [x] **Authentication:** Located the CA, Server Certificate, and Server Key in `/etc/kubernetes/pki/etcd/`.
* [x] **Execution:** Ran `etcdctl snapshot save` specifying `ETCDCTL_API=3` and the local loopback endpoint.
* [x] **Verification:** Validated the file integrity using `etcdctl snapshot status` with table output.

### 2. Live Restoration
* [x] **Extraction:** Used `etcdctl snapshot restore` to unpack the database into a safe, isolated directory (`/var/lib/etcd-backup`).
* [x] **Manifest Modification:** Edited the ETCD static pod manifest to change the `hostPath` for `etcd-data`.
* [x] **Reconvergence:** Monitored the control plane as the `kubelet` automatically restarted the ETCD pod and the API server reconnected to the restored state.