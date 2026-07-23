Here is the markdown documentation for the hardware expansion and node scheduling we just completed. You can drop this directly into your GitHub repository alongside your other guides.

### File: `docs/study-guides/11-adding-arm64-worker-node.md`

```markdown
# CKA Study Guide: Bootstrapping an ARM64 Worker Node & Node Scheduling

Adding a new worker node to an existing cluster involves configuring the OS, installing the container runtime, and using `kubeadm` to join the control plane. In a mixed-hardware environment (like adding an ARM64 Orange Pi to an AMD64 Proxmox cluster), this also unlocks architecture-specific workload scheduling.

## Phase 1: Operating System Preparation

Before Kubernetes can run, the node's underlying Linux environment must be properly configured.

**1. Disable Swap & Configure Local DNS:**
Kubernetes requires swap memory to be disabled so the Kubelet can accurately manage resources. We also map the hostname to the local loopback to prevent `kubeadm` pre-flight warnings.
```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
echo "127.0.1.1 <your-node-hostname>" | sudo tee -a /etc/hosts

```

**2. Load Kernel Modules & Configure Networking:**
Required for `kube-proxy` and the CNI (e.g., Calico or Flannel) to route traffic properly.

```bash
cat <<EOF ## 'deb 's/SystemdCgroup="false/SystemdCgroup" (Containerd) (Run (`kubeadm`, *(Note: **1. **2. **3. --dearmor --print-join-command --system -fsSL -i -o -p -y /' /etc/apt/keyrings /etc/apt/keyrings/kubernetes-apt-keyring.gpg /etc/apt/sources.list.d/kubernetes.list /etc/containerd /etc/containerd/config.toml /etc/modules-load.d/k8s.conf /etc/sysctl.d/k8s.conf 2: 3: 4: <<EOF <control-plane-ip CA Cluster Command Components Conntrack:** Container Control EOF Ensure Execute Generate Install Join Kubelet's Kubernetes Linux Minimal Missing New Node):** Paste Phase Plane):** Runtime Worker [signed-by="/etc/apt/keyrings/kubernetes-apt-keyring.gpg]" ``` ```bash `conntrack`, `kube-proxy` `kubeadm `kubectl`) `kubelet`, `sudo` `systemd` a and apt-get apt-mark apt-transport-https authenticity binaries bootstrap br_netfilter ca-certificates cat cause cgroup cluster cluster).* config configuration. configure connections. conntrack container containerd control create curl data. default distributions driver, during echo enable error explicitly fatal from gpg hash hold [https://pkgs.k8s.io/core:/stable:/v1.31/deb/](https://pkgs.k8s.io/core:/stable:/v1.31/deb/) [https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key](https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key) install it join join` join`. key kubeadm kubectl kubelet lack major/minor match matching mkdir modprobe necessary net.bridge.bridge-nf-call-ip6tables="1" net.bridge.bridge-nf-call-iptables="1" net.ipv4.ip_forward="1" of often on output overlay plane. pre-flight previous privileges. relies rest restart runtime sed step sudo sysctl systemctl tee the this to token track true/g' update use uses verify versions which will with your |>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>

```

Verify the node has joined by running `kubectl get nodes -w` on the control plane.

---

## Phase 5: Hardware-Specific Scheduling (`nodeSelector`)

When dealing with a multi-architecture cluster, `k3s` and `kubeadm` automatically apply architecture labels to nodes upon initialization (e.g., `kubernetes.io/arch=arm64` or `kubernetes.io/arch=amd64`).

To force a deployment strictly onto ARM64 hardware (preventing it from scheduling on x86 nodes), use a `nodeSelector` in the Pod spec:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: arm-only-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: arm-test
  template:
    metadata:
      labels:
        app: arm-test
    spec:
      nodeSelector:
        kubernetes.io/arch: arm64
      containers:
      - name: web-server
        image: nginx:alpine

```

```

You can commit this right into your repo. Have a good break, and we will pick up the Network Policy firewall challenge next time!

```