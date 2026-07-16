# CKA Study Guide: Domain 3 - Services & Networking

Networking in Kubernetes represents the abstraction layer that enables dynamic, ephemeral pods to be reached reliably both within the cluster and from the external world. This domain focuses on the mechanics of service discovery, routing rules, and the components that drive them.

## Core Architecture: Kube-Proxy

Kube-proxy is a daemon that runs on every node in the cluster. It is not a traditional proxy that handles application-layer traffic. Instead, it translates Kubernetes Service definitions from the API server into low-level networking rules in the host's Linux kernel (using iptables or IPVS).
*   Monitors the API Server for new Services and Endpoints.
*   Writes DNAT (Destination Network Address Translation) rules directly into the host OS.
*   Intercepts traffic bound for a Service's Virtual IP and load-balances it to the backend pods before the packet leaves the node.

## Service Types

| Type | Scope | Mechanism | Use Case |
| :--- | :--- | :--- | :--- |
| **ClusterIP** (Default) | Internal only | Assigns a Virtual IP (VIP) routable only within the cluster network. | Microservice-to-microservice communication, internal databases. |
| **NodePort** | External (Layer 4) | Builds on ClusterIP. Opens a high port (30000-32767) on the physical IP of every node. | Direct external access without an Ingress controller, or exposing an Ingress controller to a bare-metal LoadBalancer. |
| **LoadBalancer** | External (Layer 4) | Requests a public/external IP from the cloud provider (or MetalLB on bare-metal). | Exposing a service to the internet with a dedicated, standard port (e.g., 80/443). |

## Layer 7 Routing: Ingress

While NodePorts and LoadBalancers operate at Layer 4 (IP/Port), Ingress operates at Layer 7 (HTTP/HTTPS), allowing intelligent routing based on URLs and hostnames.
*   **Ingress Controller:** The actual engine (e.g., NGINX) running as a pod, typically exposed via MetalLB or a NodePort.
*   **Ingress Resource:** The routing rules (YAML) that tell the controller how to split traffic (e.g., routing `/api` to one service and `/web` to another).

## Internal Service Discovery: CoreDNS

CoreDNS provides name resolution inside the cluster, allowing pods to communicate using human-readable names instead of ephemeral IP addresses.
*   The Kubelet injects a custom `/etc/resolv.conf` into every pod, pointing DNS queries to the CoreDNS service.
*   **FQDN Format:** `<service-name>.<namespace>.svc.cluster.local`
*   Pods in the same namespace can resolve services using just the `<service-name>`.