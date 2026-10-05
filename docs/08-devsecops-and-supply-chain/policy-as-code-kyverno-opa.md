# Track 08 // Policy as Code: Kyverno & OPA Gatekeeper

Admission controllers act as the gatekeeper for Kubernetes API requests, validating or mutating objects before they are committed to `etcd`.

---

## 1. Kubernetes Admission Controller Lifecycle

```mermaid
sequenceDiagram
    autonumber
    participant U as User / CI Pipeline
    participant API as kube-apiserver
    participant M as Mutating Webhooks (Kyverno)
    participant V as Validating Webhooks (Kyverno / OPA)
    participant ETCD as etcd Storage

    U->>API: kubectl apply -f manifest.yaml
    API->>API: Authentication & Authorization (RBAC)
    API->>M: Mutating Webhook Call
    Note over M: Inject default labels, sidecars, runAsNonRoot
    API->>V: Validating Webhook Call
    alt Policy Passed
        V-->>API: 200 OK (Allowed)
        API->>ETCD: Persist Resource
        API-->>U: Resource created successfully
    else Policy Violation
        V-->>API: 403 Forbidden (Blocked)
        API-->>U: Error: Disallowed privileged container
    end
```

---

## 2. Kyverno ClusterPolicy: Enforce Restricted Security Standard

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-privileged-and-root
  annotations:
    policies.kyverno.io/title: Disallow Privileged Containers and Root User
    policies.kyverno.io/severity: high
spec:
  validationFailureAction: Enforce
  background: true
  rules:
    - name: check-privileged
      match:
        any:
          - resources:
              kinds:
                - Pod
      validate:
        message: "Privileged containers and root execution are forbidden in this cluster."
        pattern:
          spec:
            =(securityContext):
              =(runAsNonRoot): true
            containers:
              - =(securityContext):
                  =(privileged): false
                  =(allowPrivilegeEscalation): false
```
