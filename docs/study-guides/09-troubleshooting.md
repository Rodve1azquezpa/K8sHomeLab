# CKA Study Guide: Node & Cluster Troubleshooting

Troubleshooting accounts for 30% of the CKA exam. When a node or component fails, always troubleshoot from the bottom up: **OS (Systemd) -> Runtime (CRI) -> Kubernetes (Kubelet).**

## Pillar 1: Container Runtime Interface (CRI)

When a node goes `NotReady`, the connection between the Kubelet and the container runtime (`containerd`) is usually broken.

### Triage Commands
*   **Check Runtime Engine:** `sudo systemctl status containerd`
*   **Test CRI Socket:** `sudo crictl info` (If this fails, the Kubelet cannot talk to containerd).
*   **Clear Deprecated Warnings:** Ensure `crictl` knows where to look by creating `/etc/crictl.yaml` containing:
    `runtime-endpoint: unix:///run/containerd/containerd.sock`

### Common Failure Points
1.  **CRI Plugin Disabled:** Check `/etc/containerd/config.toml` to ensure the `cri` plugin isn't in the disabled list.
2.  **Wrong Socket Path:** Check the dynamically generated Kubelet flags in `/var/lib/kubelet/kubeadm-flags.env` to ensure it points to the correct Unix socket.
3.  **Cgroup Mismatch:** Kubelet and Containerd MUST use the same cgroup driver (usually `systemd`).

---

## Pillar 2: Node Disk Pressure

When a node's filesystem hits capacity, the Kubelet applies a `DiskPressure` taint, blocks new pods, and aggressively evicts running workloads to survive.

### Triage Commands
*   **Check Node Conditions:** `kubectl describe node <name> | grep -A 5 Conditions`
*   **Check OS Disk:** `df -h`
*   **Clear Dead Containers:** `sudo crictl rm $(sudo crictl ps -a -q -s exited)`
*   **Prune Unused Images:** `sudo crictl rmi --prune`

### Configuration
The eviction thresholds are defined in the Kubelet's main configuration file. If a node is falsely triggering pressure, adjust these values and restart the Kubelet:
*   **File:** `/var/lib/kubelet/config.yaml`
*   **Keys:** `evictionHard: nodefs.available: "10%"`

---

## Pillar 3: Systemd & Logging

When the API server is unreachable or the Kubelet refuses to start, standard `kubectl` commands do not work. You must drop down to the OS system daemon.

### Triage Commands
*   **Check Service Status:** `sudo systemctl status kubelet`
*   **Read Daemon Logs:** `sudo journalctl -u kubelet -n 20 --no-pager`

### The Systemd Hierarchy
Systemd loads configuration from two places. Knowing the difference is critical for the exam:
1.  `/usr/lib/systemd/system/`: Vendor defaults installed by the package manager (`apt`). 
    *   *Example:* `/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf`
2.  `/etc/systemd/system/` (or `/etc/default/`): Administrator overrides. Files here take precedence over vendor defaults.
    *   *Example:* Adding `KUBELET_EXTRA_ARGS="--custom-flag"` to `/etc/default/kubelet`.

**Crucial Step:** Whenever you modify a systemd file, you must rebuild the dependency tree before restarting the service:
```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet