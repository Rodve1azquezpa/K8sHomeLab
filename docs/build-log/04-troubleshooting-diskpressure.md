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