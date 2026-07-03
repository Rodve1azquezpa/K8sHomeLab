# Phase 2: Cluster Bootstrap & Storage Integration Status

## 🏗️ Deployment Overview
The bare-metal Proxmox hypervisor has been successfully configured as an NFS storage backend, exposing a 2TB physical USB drive. The Kubernetes cluster has been initialized, networked with Calico, and successfully bound to the physical storage.

| Component | Status | Details |
| :--- | :--- | :--- |
| **Control Plane** | 🟢 Ready | Initialized with `kubeadm`. `kubectl` admin config applied. |
| **Worker Nodes** | 🟢 Ready | `k8s-worker1` and `k8s-worker2` successfully joined to the cluster. |
| **CNI (Network)** | 🟢 Active | Calico operator and CRDs deployed. Pod overlay network active. |
| **Proxmox Storage** | 🟢 Active | Enterprise repos disabled; `nfs-kernel-server` running. Exporting `/mnt/pve/media-drive`. |
| **K8s Storage** | 🟢 Bound | `pv-media-2tb` (1800Gi) created and successfully bound to `pvc-media`. |

---

## ✅ Task Completion Checklist

### 1. Cluster Initialization & Networking
* [x] **Initialize Control Plane:** Executed `kubeadm init --pod-network-cidr=192.168.0.0/16`.
* [x] **CNI Deployment:** Deployed Calico as the Container Network Interface.
* [x] **Node Attachment:** Executed `kubeadm join` on both worker VMs. 

### 2. Physical Storage Backend (Proxmox)
* [x] **Repository Fix:** Switched Proxmox `apt` repositories from Enterprise to `no-subscription` to resolve 401 Unauthorized errors.
* [x] **NFS Server:** Installed `nfs-kernel-server` and configured `/etc/exports` with `no_root_squash` to allow Kubernetes permission management.
* [x] **Client Tools:** Installed `nfs-common` on all Kubernetes nodes to enable volume mounting.

### 3. Kubernetes Storage Primitives
* [x] **PersistentVolume (PV):** Created cluster-level resource pointing to the Proxmox NFS server IP.
* [x] **PersistentVolumeClaim (PVC):** Claimed the 1800Gi capacity for future Media Server workloads.
* [x] **End-to-End Validation:** Imperatively generated a Pod manifest, attached the PVC, and successfully wrote data from the container to the physical USB drive.