# Phase 1: Infrastructure Preparation Status

## 🏗️ Architecture Overview

The foundational virtual hardware has been provisioned on the Dell Tower hypervisor (Proxmox VE). The environment consists of three Ubuntu VMs allocated to support the Kubernetes control plane and worker workloads.

| Node Name | Role | vCPU | RAM | Disk | IP Address | Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `k8s-control` | Control Plane | 2 | 4 GB | 30 GB | *(See network-architecture.md)* | 🟢 Active |
| `k8s-worker1` | Primary Worker | 2 | 4 GB | 30 GB | *(See network-architecture.md)* | 🟢 Active |
| `k8s-worker2` | Secondary Worker | 2 | 4 GB | 30 GB | *(See network-architecture.md)* | 🟢 Active |

---

## ✅ Task Completion Checklist

All pre-requisite configurations required for a CKA-compliant Kubernetes cluster have been successfully applied across all three nodes.

### 1. Operating System Configuration
* [x] **Disable Swap:** Swap partitioned turned off and removed from `/etc/fstab` to ensure `kubelet` stability.
* [x] **Kernel Modules:** Loaded `overlay` and `br_netfilter` for container networking.
* [x] **Network Routing:** Configured `sysctl` to enable IPv4 forwarding and allow `iptables` to process bridged traffic.

### 2. Container Runtime Installation
* [x] **Runtime:** Installed `containerd.io` from official Docker repositories.
* [x] **Configuration:** Generated default `config.toml`.
* [x] **Cgroup Driver:** Configured `SystemdCgroup = true` to align with the kubelet's cgroup management.

### 3. Kubernetes Components (v1.30)
* [x] **Installation:** Added `pkgs.k8s.io` repositories and installed `kubeadm`, `kubelet`, and `kubectl`.
* [x] **Version Locking:** Applied `apt-mark hold` to prevent unintended OS updates from breaking cluster compatibility.
* [x] **Service:** Enabled `kubelet` to start on boot.

---

## ⏭️ Next Steps (Pending)
* Initialize the Control Plane using `kubeadm init`.
* Select and deploy a Container Network Interface (CNI) plugin.
* Join worker nodes to the cluster.