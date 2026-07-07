Excellent work today. You successfully diagnosed a live cluster failure, identified the resource constraint, and re-architected the storage footprint to fix it. That is exactly what a certified administrator does.

Here are your final Git repository updates to document today’s troubleshooting session. You can drop these right into your `docs/` folders.

---

### File: `docs/build-log/04-troubleshooting-diskpressure.md`

```markdown
# Phase 4: Troubleshooting & Storage Re-Architecture

## 🚨 Incident Overview
Jellyfin pods entered a `Failed` state with an `Evicted` reason during media playback. 

| Component | Status | Diagnosis |
| :--- | :--- | :--- |
| **Worker Nodes** | ⚠️ Stressed | `k8s-worker1` reported a `DiskPressure` condition. |
| **Jellyfin Pod** | 🔴 Evicted | Video transcoding wrote massive ephemeral chunks to the container's root file system, filling the VM's 30GB virtual disk. |
| **Resolution** | 🟢 Stable | Re-architected the Deployment to map the `/cache` directory directly to the physical 2TB NFS drive via `subPath`. |

---

## ✅ Task Completion Checklist

### 1. Diagnosis & Triage
* [x] **Pod Inspection:** Used `kubectl describe pod <pod-name>` to identify the `DiskPressure` rejection message.
* [x] **State Cleanup:** Executed `kubectl delete pods --field-selector status.phase=Failed` to bulk-clear the dead pod autopsy reports from the cluster state.

### 2. Architecture Remediation
* [x] **Deployment Update:** Added a third `subPath` (`jellyfin-cache`) mapping the container's `/cache` directory to the `pvc-media` volume claim.
* [x] **Live Rollout:** Applied the YAML; the Deployment controller gracefully terminated the failing pod and spun up the corrected configuration.
* [x] **Validation:** Video transcoding now successfully bypasses the node's local disk, utilizing the network-attached storage backend.

```

---

### File: `docs/study-guides/04-evictions-and-ephemeral-storage.md`

# CKA Study Guide: Pod Evictions & Ephemeral Storage

## 1. Node Conditions & The Kubelet

The `kubelet` acts as the guardian of the physical node. It constantly monitors local resources. If a node begins to run out of capacity, the `kubelet` sets a Node Condition to `True`.

### Common Pressure Conditions (Must Memorize):

| Condition | Trigger | Result |
| --- | --- | --- |
| **DiskPressure** | The root filesystem or image filesystem is almost full. | Kubelet attempts to garbage collect unused images, then evicts pods. |
| **MemoryPressure** | The node is running out of physical RAM. | Kubelet evicts pods, usually starting with those lacking resource requests/limits. |
| **PIDPressure** | Too many processes are running on the Linux host. | Kubelet evicts pods to prevent the kernel from locking up. |

## 2. Ephemeral vs. Persistent Storage

In Kubernetes, by default, anything a container writes to its own file system is **ephemeral**.

* **The Trap:** Ephemeral storage lives on the worker node's actual hard drive (in `/var/lib/containerd/...`). If a workload generates heavy temporary files (like video transcoding, log generation, or database sorting), it will quickly exhaust the node's disk, causing `DiskPressure`.
* **The Solution:** Any directory that expects heavy read/write operations must be mapped to a `PersistentVolume` (like NFS, EBS, or Ceph).

## 3. Crucial CKA Troubleshooting Commands

When pods are failing, rely on this specific order of operations:

1. `kubectl get pods` (Identify the state: CrashLoopBackOff, Evicted, Pending).
2. `kubectl describe pod <name>` (Look at the `Events` section at the very bottom for scheduler or volume mount failures).
3. `kubectl logs <name>` (Look for application-level crashes).
4. `kubectl logs <name> --previous` (Crucial if the pod is constantly restarting; tells you why the *last* instance died).
5. `kubectl get events --sort-by='.metadata.creationTimestamp'` (A cluster-wide timeline of everything that just happened).

---

Take a well-deserved break. When you are ready for the next session, we will shift focus to **Domain 2: Workloads & Scheduling**. We will boot up that Raspberry Pi, tackle ARM64 multi-architecture networking, and use Taints and Tolerations to strictly control where your workloads are allowed to run.