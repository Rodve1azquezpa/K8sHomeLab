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