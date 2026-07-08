# Phase 5: Hybrid Bare-Metal Expansion & Scheduling Status

## 🏗️ Deployment Overview
The cluster has been expanded beyond the virtualized Proxmox environment to include physical hardware. A Dell Netbook (AMD A9, 4GB RAM) was introduced as a physical edge node. Advanced scheduling constraints were applied to protect the low-resource hardware from heavy workloads.

| Node Name | Role | Architecture | Status | Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `k8s-control` | Control Plane | `amd64` (VM) | 🟢 Ready | Default Control Plane Taint |
| `k8s-worker1` | Heavy Worker | `amd64` (VM) | 🟢 Ready | None |
| `k8s-worker2` | Heavy Worker | `amd64` (VM) | 🟢 Ready | None |
| `k8s-netbook` | Edge Worker | `amd64` (Bare Metal)| 🟢 Ready | Tainted: `hardware=low-resource` |

---

## ✅ Task Completion Checklist

### 1. Physical Node Bootstrapping
* [x] **OS Preparation:** Installed Ubuntu Server 24.04 LTS on the Dell Netbook and disabled swap memory.
* [x] **Cluster Join:** Generated a new `kubeadm` token and successfully joined the physical node to the virtualized control plane.

### 2. Diagnostics & Infrastructure Fixes
* [x] **State Diagnosis:** Identified Jellyfin pod stuck in `ContainerCreating` on the new node.
* [x] **Event Tracing:** Used `kubectl describe pod` to trace the failure to a missing `nfs-common` package on the host OS, preventing PVC mounting.
* [x] **Remediation:** Installed the missing NFS client tools on the Netbook, allowing the pod to successfully transition to `Running`.

### 3. Advanced Scheduling
* [x] **Taint Application:** Applied `kubectl taint nodes k8s-netbook hardware=low-resource:NoSchedule` to prevent default pod scheduling.
* [x] **Eviction & Validation:** Imperatively deleted the active Jellyfin pod. Validated that the Deployment controller successfully recreated the pod, and the scheduler correctly bypassed the tainted Netbook to place it on a Proxmox VM.