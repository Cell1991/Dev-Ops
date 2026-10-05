# Track 10 // Chaos Engineering & Resiliency Validation

Chaos Engineering is the discipline of experimenting on a system in order to build confidence in the system's capability to withstand turbulent conditions in production.

---

## 1. Principles of Chaos Experimentation

```mermaid
graph LR
    H[1. Define Steady State] --> INJECT[2. Inject Chaos Experiment]
    INJECT --> OBS[3. Observe System Metrics]
    OBS --> VERIFY[4. Verify Hypotheses & Auto-Recovery]
    VERIFY --> FIX[5. Fix Weaknesses Found]
```

---

## 2. Chaos Mesh Experiment: Network Latency & Packet Loss

```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: simulate-cross-az-latency
  namespace: production
spec:
  action: delay
  mode: fixed
  value: '2'
  selector:
    namespaces:
      - production
    labelSelectors:
      app: payment-service
  delay:
    latency: '150ms'
    jitter: '20ms'
    correlation: '50'
  duration: '10m'
  scheduler:
    cron: '0 14 * * 2' # Run every Tuesday at 14:00
```
