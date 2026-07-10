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