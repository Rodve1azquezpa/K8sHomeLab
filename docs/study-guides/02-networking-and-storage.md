Here are the summaries and study materials to document your progress and solidify the concepts for the CKA exam.

You can save the first block into your `docs/build-log/` and the second block into a new `docs/study-guides/` directory in your Git repository.

---

### File: `docs/build-log/02-cluster-bootstrap-and-storage.md`

```markdown
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

```

---

### File: `docs/study-guides/02-networking-and-storage.md`

# CKA Study Guide: Cluster Networking & Persistent Storage

## 1. Cluster Initialization & The Pod Overlay

When building a cluster, `kubeadm` configures the control plane components (API server, etcd, scheduler, controller-manager) as static pods.

* **The `--pod-network-cidr` Flag:** As an SP mobility engineer, you are familiar with allocating IP subnets for mobile subscribers. In Kubernetes, every Pod gets its own IP address. This flag reserves a massive CIDR block (like `/16`) exclusively for the internal container network.
* **The CNI (Calico):** Kubernetes does not provide network routing out of the box; it relies on third-party plugins (CNIs). Calico uses standard networking protocols (BGP) to distribute routing information about which Pod IPs live on which physical nodes. Without a CNI, your nodes will forever remain in a `NotReady` state.

## 2. The Kubernetes Storage Paradigm

Kubernetes handles storage by separating the infrastructure details from the application details. This is tested heavily in **CKA Domain 3 (Storage)**.

### A. PersistentVolume (PV) - *The Administrator's Job*

* **Concept:** A PV is a cluster-level resource. It represents a piece of physical storage (like your NFS share, an AWS EBS volume, or a local disk).
* **Key Fields:** `capacity`, `accessModes`, `persistentVolumeReclaimPolicy` (Retain/Delete/Recycle), and the backend details (e.g., `nfs.server` and `nfs.path`).
* **Exam Note:** PVs exist independently of any namespace.

### B. PersistentVolumeClaim (PVC) - *The Developer's Request*

* **Concept:** A PVC is a request for storage by a user. It asks for a specific size and access mode. Kubernetes acts as a matchmaker, finding a PV that satisfies the PVC's requirements and binding them together.
* **Exam Note:** PVCs are bound to a specific namespace. A Pod in Namespace A cannot use a PVC in Namespace B.

### C. Access Modes Explained

* **ReadWriteOnce (RWO):** Can be mounted as read-write by a single node. (Common for databases).
* **ReadWriteMany (RWX):** Can be mounted as read-write by multiple nodes simultaneously. (Required for NFS/NAS setups where multiple pods across different nodes need to write to the same file share).
* **ReadOnlyMany (ROX):** Can be mounted read-only by multiple nodes.

## 3. Exam Survival Skills: Imperative Speed & YAML Troubleshooting

### A. The "Dry-Run" Strategy

You cannot create PVs or PVCs imperatively, but you *must* create Pods and Deployments imperatively to save time.

* **The Command:** `kubectl run <name> --image=<image> --dry-run=client -o yaml > file.yaml`
* **The Benefit:** This generates 80% of the required YAML instantly, free of syntax errors. You then open the file and append only the complex parts (like `volumeMounts` and `volumes`).

### B. Conquering YAML Indentation

YAML relies strictly on spacing (never tabs). The error `mapping values are not allowed in this context` means a child element is incorrectly indented relative to its parent.

* **Example of Failure:**
```yaml
volumeMounts:
- name: data
    mountPath: /data  # <-- ERROR: Indented too far. Must align with 'name'

```


* **The `vim` Paste Fix:** When copying configurations from the K8s documentation during the exam, terminal editors like `vim` will often try to auto-indent your pasted text, destroying the YAML structure. **Always run `:set paste` in vim before pasting** to preserve the exact formatting from the docs.