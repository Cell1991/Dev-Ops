# Track 05 // Multi-Tenancy & Ephemeral Preview Environments

Ephemeral environments provide disposable, isolated, production-like staging environments created dynamically on Pull Request creation and destroyed on PR merge.

---

## 1. Virtual Clusters (vCluster) Architecture

```mermaid
graph TD
    subgraph Host Kubernetes Cluster
        NODE[Host Physical / Cloud Nodes]
        CNI[Shared Host CNI & Storage]
        
        subgraph Namespace: pr-482-tenant
            SYNCS[vCluster Syncer]
            VAPI[Virtual k8s API Server]
            VPODS[Virtual Pod Workloads]
        end
    end

    VPODS --> SYNCS
    SYNCS -->|Sync Low-Level Pod Specs| NODE
```

### Benefits:
* Developers have full `cluster-admin` privileges inside their virtual cluster without endangering the physical host cluster.
* Spins up in < 15 seconds compared to minutes for full cloud clusters.
* Significantly cuts staging infrastructure cloud spend.
