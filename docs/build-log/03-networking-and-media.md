# Phase 3: Bare-Metal Networking & Media Server Status

## 🏗️ Deployment Overview
The Kubernetes cluster has been successfully bridged to the physical home network (`192.168.68.x`). A bare-metal load balancer (MetalLB) was deployed to handle Layer 2 ARP announcements, allowing physical network clients to seamlessly access internal Kubernetes Services. Jellyfin was deployed as the primary media server utilizing the NFS persistent storage.

| Component | Status | Details |
| :--- | :--- | :--- |
| **MetalLB Controller** | 🟢 Active | Running in `metallb-system` namespace. |
| **IP Address Pool** | 🟢 Active | `192.168.68.200 - 192.168.68.240` allocated to Kubernetes. |
| **L2 Advertisement** | 🟢 Active | MetalLB speaker pods announcing IPs via ARP to the home router. |
| **Jellyfin Deployment** | 🟢 Active | Running 1 replica, restricted to `amd64` architecture nodes. |
| **Jellyfin Service** | 🟢 Active | Type: `LoadBalancer`. Assigned VIP: `192.168.68.200`. |
| **Storage SubPaths** | 🟢 Active | `/config` and `/media` correctly mapped to physical USB subdirectories. |

---

## ✅ Task Completion Checklist

### 1. Bare-Metal Load Balancing
* [x] **Router Configuration:** DHCP pool restricted to end at `.199` to prevent IP conflicts.
* [x] **MetalLB Installation:** Applied native manifests to create the controller and speaker DaemonSets.
* [x] **IP Pool & L2 Config:** Created custom resources to define the `.200-.240` range and enabled Layer 2 advertisement.
* [x] **Validation Test:** Imperatively deployed an NGINX web server, exposed it as a LoadBalancer, and successfully accessed it from a physical laptop.

### 2. Media Server Deployment
* [x] **Manifest Creation:** Wrote a multi-resource YAML containing a Deployment and a Service.
* [x] **Node Selection:** Applied `nodeSelector` to ensure the media server only runs on `amd64` nodes (preparing for ARM64 Pi integration).
* [x] **Advanced Storage (`subPath`):** Mounted the same PVC to two different container directories (`/config` and `/media`) to separate application data from user media.