CKA Study Guide: Services & Networking
1. The Deployment Controller (Self-Healing)
A Deployment does not run containers; it manages ReplicaSets, which in turn manage Pods.

The Concept: You declare a "desired state" (e.g., "I want 3 NGINX pods"). The Deployment's control loop constantly measures the "actual state." If a node crashes and takes a pod with it, the actual state becomes 2. The Deployment immediately spins up a new pod on a healthy node to restore the state to 3.

Exam Tip: In the CKA, you will often be asked to scale a deployment. Do this imperatively: kubectl scale deployment <name> --replicas=5.

2. Kubernetes Services (The Stable VIP)
Because Pod IPs are highly ephemeral (changing every time a pod dies and is recreated), we need a stable entry point. This is the Service. Services use label selectors to know which pods to send traffic to.

Service Types (Must Memorize):
ClusterIP (Default): Accessible only from inside the cluster. Used for internal microservice communication (e.g., a frontend pod talking to a backend database pod).

NodePort: Opens a static port (30000-32767) on the IP address of every worker node in the cluster.

LoadBalancer: Integrates with an external load balancer (like AWS ELB, or MetalLB on bare metal) to assign a dedicated, routable IP address.

How iptables powers Services:
As a network engineer, it is crucial to understand that a Service is not a physical proxy or a pod. It is literally just a set of iptables (or IPVS) rules injected into the Linux kernel of every worker node by the kube-proxy component. When a packet hits the Node's interface destined for the Service IP, iptables intercepts it and DNATs (Destination Network Address Translation) it to a healthy Pod's IP.

3. Storage: The subPath Directive
In CKA Domain 3 (Storage), you will encounter scenarios where a pod needs to mount multiple directories, but you only have one Persistent Volume Claim.

The Problem: If you mount pvc-media to /config and /media, the container will just see the exact same files in both directories.

The Solution: subPath. This tells Kubernetes to create a specific subdirectory inside the physical volume, and only mount that specific subdirectory into the container's path.

🚀 What's Next? (Phase 4)
We now have a highly functional amd64 cluster. To truly prepare you for the CKA, we need to introduce scheduling complexity.

Next Objective: Multi-Architecture & Advanced Scheduling (CKA Domain 2)
I propose we power up your Raspberry Pi (arm64) or your Dell Netbook (AMD A9 - 4GB RAM).

Once joined to the cluster, we will practice:

Taints & Tolerations: How to "repel" heavy workloads (like Jellyfin) from deploying on the weak Netbook.

Node Affinity: How to attract specific workloads (like a lightweight DNS sinkhole, e.g., Pi-hole) strictly to the Raspberry Pi.

Which device would you like to bootstrap and join to the cluster next? The Pi or the Netbook?