# Track 06 // Service Mesh, Envoy & Zero-Trust Architecture

A Service Mesh provides transparent, infrastructure-level mutual TLS (mTLS), fine-grained Layer 7 traffic routing, fault injection, and distributed telemetry.

---

## 1. Sidecar vs Ambient Mesh Architecture

```mermaid
graph TD
    subgraph Sidecar Model - High Resource Footprint
        POD1[App Container] <--> PROXY1[Envoy Sidecar Container]
        POD2[App Container] <--> PROXY2[Envoy Sidecar Container]
        PROXY1 <==>|mTLS Wire| PROXY2
    end

    subgraph Ambient Mesh - Sidecarless Architecture
        APP1[App Pod] <--> ZT1[Node ztunnel DaemonSet - L4 mTLS]
        APP2[App Pod] <--> ZT2[Node ztunnel DaemonSet - L4 mTLS]
        ZT1 <==>|HBONE Protocol / mTLS| ZT2
        ZT1 -.->|L7 Complex Policy| WP[Waypoint Proxy - Envoy L7]
    end
```

---

## 2. Zero-Trust `AuthorizationPolicy` Example

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: strict-payment-access
  namespace: production
spec:
  selector:
    matchLabels:
      app: payment-service
  action: ALLOW
  rules:
    - from:
        - source:
            principals: ["cluster.local/ns/production/sa/checkout-service-sa"]
      to:
        - operation:
            methods: ["POST"]
            paths: ["/v1/charge"]
```
