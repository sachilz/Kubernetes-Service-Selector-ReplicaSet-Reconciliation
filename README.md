# Kubernetes Cluster Operations Lab

### 🚀 Hands-on Kubernetes Architecture, Workloads, Networking, Self-Healing & Troubleshooting

[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.30+-326CE5?logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Minikube](https://img.shields.io/badge/Minikube-Local_Cluster-13B5EA?logo=kubernetes&logoColor=white)](https://minikube.sigs.k8s.io/)
[![Docker](https://img.shields.io/badge/Docker-Container_Runtime-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![WSL2](https://img.shields.io/badge/WSL2-Ubuntu_Linux-E95420?logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A practical, beginner-friendly lab guide built on a local **Minikube** cluster on **WSL2** to explore core Kubernetes architecture, workload scheduling, service discovery, dynamic scaling, pod self-healing, and real-world incident troubleshooting.

---

## 🏛️ Lab Architecture & Traffic Flow

```text
 [ Local Developer Machine / Browser ]
                  │
                  │ (HTTP Traffic to localhost:8080 via kubectl port-forward or NodePort 30080)
                  ▼
┌────────────────────────────────────────────────────────────────────────┐
│ Minikube Kubernetes Node                                               │
│                                                                        │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │ static-website-service (NodePort: 30080 | ClusterIP: 10.104.x.x)  │  │
│  │ Selector: app=static-website                                     │  │
│  └──────────────────────────────────┬───────────────────────────────┘  │
│                                     │                                  │
│                       Dynamic Traffic Load Balancing                   │
│                       via EndpointSlices (Port 80)                     │
│                                     │                                  │
│         ┌───────────────────────────┼───────────────────────────┐      │
│         ▼                           ▼                           ▼      │
│  ┌───────────────┐           ┌───────────────┐           ┌──────────┐  │
│  │ Pod Replica 1 │           │ Pod Replica 2 │           │ Pod Rep 3│  │
│  │ 10.244.0.2:80 │           │ 10.244.0.3:80 │           │ ...      │  │
│  │ static-website│           │ static-website│           │ static-..│  │
│  └───────────────┘           └───────────────┘           └──────────┘  │
│                                                                        │
│ ────────────────────────────────────────────────────────────────────── │
│ Control Plane Components (kube-system Namespace)                       │
│ ┌───────────────┐  ┌───────────────┐  ┌───────────────┐  ┌───────────┐ │
│ │ kube-apiserver│  │     etcd      │  │kube-scheduler │  │ kube-ctrl │ │
│ └───────────────┘  └───────────────┘  └───────────────┘  └───────────┘ │
│ ┌───────────────┐  ┌───────────────┐  ┌───────────────┐                │
│ │    CoreDNS    │  │  kube-proxy   │  │    kindnet    │                │
│ └───────────────┘  └───────────────┘  └───────────────┘                │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 📚 Step-by-Step Learning Track

Follow these modules in numerical order to master Kubernetes cluster operations:

| Step | Guide & Module | Topics Covered | Key Commands |
| :---: | :--- | :--- | :--- |
| **01** | [**01 - Basic Cluster Inspection**](01-basic-cluster-inspection/kubectl-commands.md) | Cluster reachability, control plane endpoints, node statuses, OS/kernel versions, and global pod listings. | `kubectl cluster-info`<br>`kubectl get nodes -o wide`<br>`kubectl get pods -A -o wide` |
| **02** | [**02 - Workload & System Inspection**](02-workload-inspection/workload-service-inspection.md) | Deep-dive into CoreDNS diagnostics, system control-plane pods (`etcd`, `apiserver`), ReplicaSets, and Deployments. | `kubectl describe pods coredns`<br>`kubectl get pods -n kube-system`<br>`kubectl describe deployment` |
| **03** | [**03 - Service Networking & Endpoints**](03-service-networking/service-endpoints-inspection.md) | How Services map ports (`ClusterIP`, `NodePort`, `TargetPort`), discovery mechanics, and routing to Pod IPs. | `kubectl describe svc`<br>`kubectl get endpoints`<br>`kubectl get svc -o yaml` |
| **04** | [**04 - Workload Scaling & Port Forwarding**](04-scaling-and-access/scaling-and-port-forwarding.md) | Local development tunneling via `port-forward`, scaling replicas dynamically, and tracking modern EndpointSlices. | `kubectl port-forward svc 8080:80`<br>`kubectl scale deployment --replicas=3`<br>`kubectl get endpointslices` |
| **05** | [**05 - Self-Healing & Pod Lifecycle**](05-self-healing/pod-self-healing.md) | Cluster auto-recovery: manual pod termination, watching live ReplicaSet healing in real-time, and traffic rerouting. | `kubectl delete pod <name>`<br>`kubectl get pods -w`<br>`kubectl get deployment` |
| **06** | [**06 - Service Discovery & Label Troubleshooting**](06-service-discovery-troubleshooting/service-label-matching.md) | Debugging Service-to-Pod connectivity, label selector typos, inspecting EndpointSlice YAML conditions, and `jsonpath`. | `kubectl get pods --show-labels`<br>`kubectl get endpointslices -o yaml`<br>`kubectl get svc -o jsonpath` |
| **07** | [**07 - Pod Quarantine & Isolation for Debugging**](07-pod-isolation-and-debugging/pod-quarantine.md) | Isolating a failing pod from live traffic for offline debugging while the Deployment automatically spawns a healthy replacement. | `kubectl label pod <name> app=test --overwrite`<br>`kubectl get pods --show-labels` |
| **08** | [**08 - Pod Re-integration & Over-Replication Reconciliation**](08-pod-reintegration/pod-reintegration-reconciliation.md) | Re-integrating an isolated pod back to production and watching the ReplicaSet automatically prune surplus replicas. | `kubectl label pod <name> app=prod --overwrite`<br>`kubectl get pods -w` |

---

## 🛠️ Technology Stack Matrix

| Layer | Technology | Purpose in Lab |
| :--- | :--- | :--- |
| **Orchestration** | **Kubernetes (v1.30+)** | Container orchestration, declarative API, self-healing, and workload scheduling. |
| **Local Cluster** | **Minikube** | Single-node virtualized Kubernetes environment running inside Docker. |
| **Operating System** | **Ubuntu 24.04 (WSL2)** | Linux subsystem hosting development tools, Docker engine, and `kubectl`. |
| **Container Engine** | **Docker Desktop & containerd** | Image builds, container lifecycle management, and runtime backing Minikube. |
| **Workload App** | **Static Website (Nginx)** | Multi-replica lightweight web service used to explore scaling and routing. |
| **DNS & Discovery** | **CoreDNS & EndpointSlices** | Cluster-internal domain resolution and scalable endpoint management. |

---

## 📂 Repository Structure

```text
kubernetes-cluster-operations/
├── README.md                                    # Root project documentation and curriculum index
│
├── 01-basic-cluster-inspection/                 # Module 01: Core Cluster Inspection
│   └── kubectl-commands.md                      # Control-plane reachability, node stats, pod listings
│
├── 02-workload-inspection/                      # Module 02: Workload & System Component Inspection
│   └── workload-service-inspection.md           # CoreDNS, system pods, ReplicaSets, and Deployments
│
├── 03-service-networking/                       # Module 03: Service Networking & Port Mappings
│   └── service-endpoints-inspection.md          # ClusterIP, NodePort, TargetPort, and Endpoints
│
├── 04-scaling-and-access/                       # Module 04: Scaling & Port Forwarding
│   └── scaling-and-port-forwarding.md           # Local access proxy, replica scaling, EndpointSlices
│
├── 05-self-healing/                             # Module 05: Self-Healing & Pod Lifecycle
│   └── pod-self-healing.md                      # Failure simulation, watch mode, auto-recovery
│
├── 06-service-discovery-troubleshooting/        # Module 06: Label Selector Troubleshooting
│   └── service-label-matching.md                # Label matching, EndpointSlice conditions, jsonpath
│
├── 07-pod-isolation-and-debugging/              # Module 07: Incident Isolation & Quarantine
│   └── pod-quarantine.md                        # Removing a pod from traffic for offline debugging
│
└── 08-pod-reintegration/                       # Module 08: Workload Reconciliation
    └── pod-reintegration-reconciliation.md      # Pod re-integration, over-replication pruning
```

---

## ⚡ Quick Start: Running the Lab

### 1. Start Your Local Minikube Cluster
```bash
minikube start --driver=docker
kubectl cluster-info
kubectl get nodes
```

### 2. Deploy Sample Workload
```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

### 3. Verify Workload & Access
```bash
# Check running pods
kubectl get pods -l app=static-website -o wide

# Tunnel traffic locally
kubectl port-forward svc/static-website-service 8080:80
```
Open [http://localhost:8080](http://localhost:8080) in your web browser.

---

## 📖 Quick Command Cheat Sheet

| Task | Command |
| :--- | :--- |
| **Check cluster health** | `kubectl cluster-info` |
| **List nodes with OS info** | `kubectl get nodes -o wide` |
| **View all pods across namespaces** | `kubectl get pods -A -o wide` |
| **Inspect a deployment** | `kubectl describe deployment <name>` |
| **Inspect Service ports & endpoints** | `kubectl describe svc <service-name>` |
| **Forward traffic to localhost** | `kubectl port-forward svc/<service> 8080:80` |
| **Scale replicas** | `kubectl scale deployment <name> --replicas=<count>` |
| **Live watch pod events** | `kubectl get pods -l app=<label> -w` |
| **Show labels on all pods** | `kubectl get pods --show-labels` |
| **Change a pod label (quarantine)** | `kubectl label pod <pod> app=test --overwrite` |

---

## 👤 Author
* **Sachintha** — DevOps & Cloud Enthusiast
* Hands-on Kubernetes & Cloud-Native Engineering Labs
