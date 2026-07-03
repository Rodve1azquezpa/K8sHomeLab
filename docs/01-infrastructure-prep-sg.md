Here is a comprehensive, visual study guide summarizing the foundation we just laid and the core concepts required for the 2026 CKA exam. You can save this directly to your `docs/` folder as study material.

---

# CKA Study Guide: Foundation & Architecture

## 1. The 2026 CKA Exam Landscape

The CKA is a 2-hour, performance-based, open-book (official K8s docs only) exam. You will not answer multiple-choice questions; you will be dropped into live terminals across multiple clusters to build, fix, and manage infrastructure.

### Domain Weight Distribution (2026 Update)

| Domain | Weight | Focus Areas |
| --- | --- | --- |
| **Troubleshooting** | 30% | Diagnosing node failures, broken workloads, kubelet issues, and network blockages. |
| **Cluster Architecture** | 25% | `kubeadm` lifecycle, etcd backup/restore, RBAC, and highly available control planes. |
| **Services & Networking** | 22% | Ingress, Gateway API, NetworkPolicies, CoreDNS, and CNI troubleshooting. |
| **Workloads & Scheduling** | 12% | Deployments, DaemonSets, node affinity, taints, and tolerations. |
| **Storage** | 10% | PersistentVolumes (PVs), PVCs, and StorageClasses. |

---

## 2. Phase 1 Conceptual Deep Dive: Node Preparation

To pass the exam (and to maintain a stable home lab), you must understand *why* we ran the commands in Phase 1, not just how to run them.

### A. Memory Management (The Swap Restriction)

* **The Action:** `swapoff -a`
* **The Concept:** Kubernetes schedules Pods based on deterministic resource requests and limits. If a Linux node starts swapping memory to disk, the `kubelet` loses its ability to accurately measure and enforce Pod memory limits.
* **CKA Tip:** If a newly joined node constantly shows a `NotReady` status in the exam, checking if swap was re-enabled upon reboot is a primary troubleshooting step.

### B. Container Networking (The SP Mobility Bridge)

In SP mobility and CPS solutions, you are used to deep packet inspection, routing, and policy enforcement. Kubernetes handles this locally on the node using `iptables` and Linux bridges.

* **The Action:** Loading `br_netfilter` and setting `net.bridge.bridge-nf-call-iptables = 1`.
* **The Concept:** A container network (like Calico or Cilium) relies on a Layer 2 bridge interface (`cni0`). However, Kubernetes NetworkPolicies and Service load balancing (via `kube-proxy`) require Layer 3 `iptables` rules. The `br_netfilter` module forces bridged IPv4 traffic to pass through the iptables chains, allowing Kubernetes to enforce network policies and route Service IPs correctly.

### C. The Cgroup Driver Conflict

* **The Action:** Setting `SystemdCgroup = true` in containerd.
* **The Concept:** Control Groups (cgroups) are a Linux kernel feature that isolates and limits the resource usage (CPU, memory) of processes. Ubuntu uses `systemd` as its default init system and cgroup manager. If `containerd` uses a different cgroup driver (like `cgroupfs`), the node will have two separate managers fighting over resource allocation, leading to instability under load.

---

## 3. The Core Kubernetes Components

When we installed the packages via `apt`, we established the triumvirate of Kubernetes administration:

### 🧩 `kubeadm` (The Bootstrapper)

* **Role:** Automates the complex cryptographic and configuration tasks required to spin up a cluster.
* **Exam Relevance:** You will use this to upgrade clusters (`kubeadm upgrade plan`, `kubeadm upgrade apply`) and join new worker nodes.

### ⚙️ `kubelet` (The Node Agent)

* **Role:** The primary "node agent" that runs on every machine. It registers the node with the API server, watches for Pod definitions assigned to its node, and instructs `containerd` to start the actual containers.
* **Exam Relevance:** `kubelet` runs as a standard `systemd` service (not a container). If a node goes down, `systemctl status kubelet` and `journalctl -u kubelet` are your best friends.

### ⌨️ `kubectl` (The Command Line Interface)

* **Role:** The tool used to communicate with the cluster's API server.
* **Exam Relevance:** Speed is everything. You must master imperative commands (e.g., `kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml`) instead of writing YAML manifests from scratch.

---

## 4. Visualizing Your Cluster Topology

Here is the conceptual map of what we are building on your hardware over the next few phases.

```text
                        🌐 Internet (Port 80/443)
                                  │
                                  ▼
                     [ Home Router Port Forward ]
                                  │
      ┌───────────────────────────┼───────────────────────────┐
      │ Proxmox Hypervisor        │                           │
      │                           ▼                           │
      │ ┌────────────────┐ ┌────────────────┐ ┌─────────────┐ │
      │ │ k8s-control    │ │ k8s-worker1    │ │ k8s-worker2 │ │
      │ │ (Control Plane)│ │ (Media Node)   │ │ (NAS Node)  │ │
      │ │ - API Server   │ │ - Jellyfin/Plex│ │ - Nextcloud │ │
      │ │ - etcd         │ │ - Ingress      │ │             │ │
      │ └────────────────┘ └────────┬───────┘ └──────┬──────┘ │
      └─────────────────────────────┼────────────────┼────────┘
                                    │                │
                             [ NFS Server ] ◄────────┘
                             (USB HDDs 4TB)

```

[FreeCodeCamp: Kubernetes Administrator Certification Course](https://www.youtube.com/watch?v=l57xKN6OBhY)

This full-length course provides a comprehensive walkthrough of the 2026 CKA curriculum, mapping perfectly to the cluster bootstrap and administration tasks you are performing on your hypervisor.