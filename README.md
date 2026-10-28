# kubernetes-cluster-operations
Hands-on Kubernetes architecture lab using Minikube to explore cluster components, Pod lifecycle, workload scheduling, scaling, self-healing, networking, and troubleshooting.

## Step-by-Step Learning Guides

Follow these guides in numerical order to learn Kubernetes cluster operations from the ground up:

1. **[01 - Basic Cluster Inspection](01-basic-cluster-inspection/kubectl-commands.md)**
   * Verify control plane endpoints, cluster connectivity, worker node details, deployments, and pods across all namespaces.
2. **[02 - Workload & System Inspection](02-workload-inspection/workload-service-inspection.md)**
   * Deep dive into CoreDNS health, control plane infrastructure pods (`etcd`, `apiserver`), ReplicaSets, and application deployments.
3. **[03 - Service Networking & Endpoints](03-service-networking/service-endpoints-inspection.md)**
   * Understand how Services map ports (`ClusterIP`, `NodePort`, `TargetPort`) and route traffic to Pod IP Endpoints.
4. **[04 - Workload Scaling & Port Forwarding](04-scaling-and-access/scaling-and-port-forwarding.md)**
   * Test apps locally via `port-forward`, dynamically scale replicas up/down, and observe live EndpointSlice updates.
5. **[05 - Self-Healing & Pod Lifecycle](05-self-healing/pod-self-healing.md)**
   * Observe cluster auto-recovery: delete a pod, watch Kubernetes spin up a healthy replacement in 2 seconds, and reroute traffic.
6. **[06 - Service Discovery & Label Troubleshooting](06-service-discovery-troubleshooting/service-label-matching.md)**
   * Debug service-to-pod connectivity by verifying label selectors, checking pod labels, inspecting EndpointSlice YAML conditions, and using `jsonpath`.
7. **[07 - Pod Quarantine & Isolation for Debugging](07-pod-isolation-and-debugging/pod-quarantine.md)**
   * Safely detach a misbehaving pod from live traffic for offline debugging while the Deployment automatically spawns a healthy replacement.
8. **[08 - Pod Re-integration & Over-Replication Reconciliation](08-pod-reintegration/pod-reintegration-reconciliation.md)**
   * Return a quarantined pod back into the production pool and observe how Kubernetes automatically reconciles and prunes surplus replicas.
