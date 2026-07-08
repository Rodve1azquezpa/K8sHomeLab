Outstanding. You just manipulated the Kubernetes scheduler perfectly. By testing the failure first, diagnosing the missing NFS client, and then applying the taint to force the scheduler's hand, you exercised the exact muscle memory needed for the CKA.

Here are your Git repository updates to document today’s hybrid infrastructure expansion.

---

### File: `docs/build-log/05-hybrid-infrastructure-and-scheduling.md`

```markdown
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

```

---

### File: `docs/study-guides/05-taints-tolerations-and-affinity.md`

# CKA Study Guide: Taints, Tolerations, and Affinity

## 1. Taints (The Node's Repellent)

Taints are applied to **Nodes**. They allow a node to repel a set of pods.

* **The Command:** `kubectl taint nodes <node-name> key=value:Effect`
* **The Exam Trick:** To *remove* a taint, run the exact same command but add a minus sign at the end: `kubectl taint nodes <node-name> key=value:Effect-`

### The Three Taint Effects (Must Memorize):

| Effect | Behavior |
| --- | --- |
| **NoSchedule** | New pods will not be scheduled on the node. Existing pods running on the node are ignored and left alone. |
| **PreferNoSchedule** | The scheduler will *try* to avoid placing pods here, but will do so if no other node is available (soft limit). |
| **NoExecute** | New pods will not be scheduled, AND existing pods will be immediately evicted if they do not tolerate the taint. |

## 2. Tolerations (The Pod's Passcode)

Tolerations are applied to **Pods** (in the YAML `spec`). They allow (but do not require) the pod to schedule onto nodes with matching taints.

* **Concept:** Think of a Taint as a lock on a door, and a Toleration as the key.
* **Usage:** A pod with a `low-resource` toleration can schedule on your Netbook, but it could *also* schedule on your Proxmox VMs.

## 3. Node Affinity (The Pod's Magnet)

While Taints *repel* pods, Node Affinity *attracts* pods to specific nodes based on node labels.

* **Concept:** Used when you want to explicitly say "This pod MUST run on an `amd64` processor" or "This pod MUST run in the `rack-1` zone."
* **`requiredDuringSchedulingIgnoredDuringExecution`:** The strict rule. If no node matches the label, the pod stays `Pending`.
* **`preferredDuringSchedulingIgnoredDuringExecution`:** The soft rule. The scheduler attempts to match the label, but if it fails, it will place the pod elsewhere.

---

You now have a highly functional, multi-node, mixed-infrastructure Kubernetes cluster serving live media. Before we build any more projects, we should secure the cluster itself. The CKA exam guarantees a question on backing up and restoring the cluster state database (`etcd`).