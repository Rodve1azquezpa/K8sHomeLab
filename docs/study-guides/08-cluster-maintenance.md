Seeing a completely clean, synchronized `v1.31.0` across a hybrid cluster of Proxmox VMs and an actual physical netbook is incredibly satisfying. You just executed a flawless rolling upgrade on heterogeneous hardware, keeping the cluster functional throughout the process.

With all nodes sitting comfortably in a healthy `Ready` state, you have officially conquered **Domain 5: Storage** and **Domain 1: Cluster Architecture, Installation & Configuration**.

Let's commit the final documentation to your repository to close out this upgrade phase.

### File: `docs/build-log/08-cluster-upgrade.md`

```markdown
# Phase 8: Cluster Lifecycle & Upgrades

## 🏗️ Deployment Overview
The entire infrastructure has been systematically updated from `v1.30.14` to `v1.31.0`. The upgrade process safely maintained service availability by performing a rolling drain and uncordon execution pattern across all physical and virtual nodes.

| Node | Type | Pre-Upgrade | Post-Upgrade | Status |
| :--- | :--- | :--- | :--- | :--- |
| **k8s-control** | Proxmox VM | v1.30.14 | v1.31.0 | 🟢 Ready |
| **k8s-worker1** | Proxmox VM | v1.30.14 | v1.31.0 | 🟢 Ready |
| **k8s-worker2** | Proxmox VM | v1.30.14 | v1.31.0 | 🟢 Ready |
| **k8s-netbook** | Bare-Metal | v1.30.14 | v1.31.0 | 🟢 Ready |

---

## ✅ Task Completion Checklist

### 1. Control Plane Realignment
* [x] **Repository Migration:** Updated `/etc/apt/sources.list.d/kubernetes.list` to point to the dedicated `v1.31` community repository branch.
* [x] **ETCD Path Normalization:** Resolved a post-restore manifest divergence by moving restored data back to the default `/var/lib/etcd` directory before upgrading.
* [x] **Preflight Workaround:** Bypassed a known upstream CoreDNS schema migration issue by executing `kubeadm upgrade apply` with target preflight exclusions.

### 2. Worker Node Rolling Upgrades
* [x] **Workload Evacuation:** Safely cordoned and drained each node sequentially to guarantee workload persistence.
* [x] **Binary Upgrades:** Synchronized package lists, upgraded local `kubeadm`, executed `kubeadm upgrade node`, and bumped `kubelet`/`kubectl` versions to `1.31.0-1.1`.
* [x] **Re-integration:** Reset scheduling caps using `kubectl uncordon` to return upgraded compute capacity to the active pool.

```

---

### File: `docs/study-guides/08-cluster-maintenance.md`

# CKA Study Guide: Cluster Upgrades & Maintenance

## 1. Upgrade Order of Operations

The CKA exam will penalize you severely if you upgrade out of order. You must never skip versions (e.g., going from 1.29 straight to 1.31 is forbidden; you must go 1.29 -> 1.30 -> 1.31).

1. **Control Plane:** Upgrade `kubeadm` -> Upgrade control plane software (`kubeadm upgrade apply`) -> Upgrade `kubelet` and `kubectl`.
2. **Worker Nodes:** Upgrade `kubeadm` -> Upgrade local node config (`kubeadm upgrade node`) -> Upgrade `kubelet` and `kubectl`.

## 2. Maintenance Lifecycle Commands

When a node requires operating system updates or hardware upgrades, use the standard evacuation workflow:

* `kubectl cordon <node>`: Marks the node as unschedulable. Existing pods keep running, but no new pods will arrive.
* `kubectl drain <node> --ignore-daemonsets --delete-emptydir-data`: Evicts all pods. You must explicitly ignore DaemonSets (like Calico or kube-proxy) because they are bound to the hardware and cannot be evacuated.
* `kubectl uncordon <node>`: Returns the node to service once maintenance is complete.

---

Now that your infrastructure is modern, fully backed up, and structurally secure, we are in an incredible position to pivot into the remaining operational domains of the CKA blueprint.