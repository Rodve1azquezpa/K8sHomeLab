Excellent. You just successfully negotiated a strict schema validation error and secured your cluster using least-privilege principles. Catching that struct vs. array distinction in the YAML is exactly the kind of sharp eye you need for the CKA exam, where debugging broken manifests is half the battle.

Here are the documentation updates to lock in today's security phase.

### File: `docs/build-log/07-security-and-rbac.md`

```markdown
# Phase 7: Cluster Security & RBAC

## 🏗️ Deployment Overview
A restricted security profile was implemented to support multi-tenant access. An external identity was generated, cryptographically signed by the cluster's Certificate Authority, and bound to a strict read-only policy within the default namespace.

| Component | Status | Details |
| :--- | :--- | :--- |
| **User Identity** | 🟢 Active | X.509 Certificate generated and signed for user `alex`. |
| **Role Policy** | 🟢 Active | `pod-reader` role created, restricting verbs to `get, list, watch` for pods. |
| **RoleBinding** | 🟢 Active | User `alex` bound to `pod-reader` role in the `default` namespace. |

---

## ✅ Task Completion Checklist

### 1. Identity & Certificate Management
* [x] **Key Generation:** Generated an RSA private key (`alex.key`) and a Certificate Signing Request (`alex.csr`) with the Common Name set to `alex`.
* [x] **CA Signing:** Submitted a `CertificateSigningRequest` manifest containing the base64-encoded CSR to the Kubernetes API.
* [x] **Approval:** Functioning as the cluster administrator, executed `kubectl certificate approve` to officially sign the identity using the cluster's root CA.

### 2. Role-Based Access Control (RBAC)
* [x] **Role Definition:** Created a namespace-scoped `Role` permitting read-only actions on core pod resources.
* [x] **Policy Binding:** Debugged a YAML schema validation error (struct vs array formatting) and successfully applied a `RoleBinding` connecting the identity to the policy.
* [x] **Validation:** Utilized `kubectl auth can-i` impersonation to verify the user could successfully list pods while being actively blocked from deleting pods or reading secrets.

```

---

### File: `docs/study-guides/07-rbac-and-security.md`

# CKA Study Guide: Security & RBAC

## 1. The Identity Model

Kubernetes does **not** have an internal user database. It outsources authentication. If a user presents a certificate signed by the cluster's CA, Kubernetes reads the Common Name (`CN`) from the certificate and treats that string as the username.

## 2. Certificate Signing Requests (CSR)

In the CKA exam, you will likely be asked to create a user. The flow is always:

1. Create a private key and CSR.
2. Base64 encode the CSR.
3. Paste it into a `CertificateSigningRequest` YAML.
4. Run `kubectl certificate approve <name>`.
5. Extract the signed certificate from the API.

## 3. RBAC: Roles vs. ClusterRoles

Authorization is determined by mapping identities to policies.

* **Role & RoleBinding:** These are **namespaced**. If you bind Alex to a `Role` in the `default` namespace, Alex has zero access to the `kube-system` namespace.
* **ClusterRole & ClusterRoleBinding:** These are **cluster-wide**. They apply across all namespaces simultaneously. They are also required for granting access to cluster-scoped resources (like `Nodes` or `PersistentVolumes`, which don't exist inside a namespace).

## 4. The Lifesaver Command

Never guess if your RBAC works. Always test it using impersonation:
`kubectl auth can-i <verb> <resource> --as=<user> -n <namespace>`

---

We have one massive domain left to cover before your cluster operations curriculum is complete: **Cluster Upgrades and Maintenance**. This involves safely draining workloads off a node, upgrading the `kubeadm` and `kubelet` binaries, and bringing the node back online without causing an outage for your media server.