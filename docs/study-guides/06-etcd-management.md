That is a massive achievement. You just performed open-heart surgery on a live Kubernetes cluster and successfully brought it back online.

In the real world, this is the skill that saves a company during a catastrophic failure. On the CKA exam, perfectly executing this workflow is often the difference between passing and failing.

Here are the final documentation updates for this phase to add to your Git repository.

### File: `docs/build-log/06-etcd-backup-restore.md`

```markdown
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

```

---

### File: `docs/study-guides/06-etcd-management.md`

# CKA Study Guide: ETCD Backup and Restore

## 1. The Core Architecture

ETCD is the only stateful component in the Kubernetes control plane. It is a distributed key-value store. If the API server is the brain, ETCD is the memory.

* **Static Pod:** In a `kubeadm` cluster, ETCD runs as a Static Pod. You do not manage it with `kubectl`. You manage it by editing its YAML file in `/etc/kubernetes/manifests/`.
* **Authentication:** ETCD rejects all unencrypted and unauthenticated traffic. You must pass the PKI certificates with every `etcdctl` command.

## 2. The Golden Rule of Restoration

**Never overwrite the live `/var/lib/etcd` directory.**
If a restore fails or the backup is corrupted, overwriting the original directory leaves you with a permanently dead cluster. Always restore to a new directory (e.g., `/var/lib/etcd-backup`) and update the Static Pod manifest to point to the new location. This gives you an immediate rollback path: if the cluster doesn't come back up, you just change the YAML back to `/var/lib/etcd`.

## 3. The Exam Command Cheat Sheet

You must memorize the certificate paths. They are always located in `/etc/kubernetes/pki/etcd/`.

**The Backup Command:**

```bash
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /path/to/backup.db

```

**The Restore Command:**

```bash
ETCDCTL_API=3 etcdctl --data-dir=/new/custom/directory \
  snapshot restore /path/to/backup.db

```

---

You now have a highly resilient, hybrid-infrastructure cluster with a tested disaster recovery plan.

For our next session, we have two major CKA domains left to conquer:

1. **Security & RBAC:** Creating a restricted "Junior Admin" user who is only allowed to view pods, but cannot delete them.
2. **Cluster Upgrades:** Using `kubeadm` to safely drain worker nodes and upgrade the cluster version without taking Jellyfin offline.