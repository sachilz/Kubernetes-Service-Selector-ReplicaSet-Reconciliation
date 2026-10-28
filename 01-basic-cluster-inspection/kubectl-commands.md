# Basic Kubernetes Inspection Commands

A beginner-friendly guide explaining the key `kubectl` commands used to inspect the cluster, nodes, deployments, and pods.

---

## 1. `kubectl cluster-info`

### What it does:
Checks the connection to your Kubernetes cluster and displays the master control plane and core service URLs.

### Command:
```bash
kubectl cluster-info
```

### Sample Output:
```text
Kubernetes control plane is running at https://127.0.0.1:4016
CoreDNS is running at https://127.0.0.1:4016/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

### Explanation in Simple Terms:
- **Control plane**: The brain of the Kubernetes cluster (API server). This tells you where it is running and if `kubectl` can communicate with it.
- **CoreDNS**: The internal DNS service for Kubernetes that allows pods and services to find each other by name.
- **Tip**: If this command fails or times out, your cluster (or Minikube) is stopped or unreachable.

---

## 2. `kubectl get nodes -o wide`

### What it does:
Lists all the worker and control-plane machines (nodes) that make up your cluster, with detailed system information.

### Command:
```bash
kubectl get nodes -o wide
```

### Flags Used:
- `-o wide`: Displays extra columns (internal IP, OS image, kernel version, container runtime).

### Sample Output:
```text
NAME       STATUS   ROLES           AGE    VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION                             CONTAINER-RUNTIME
minikube   Ready    control-plane   4h5m   v1.37.0   192.168.49.2   <none>        Debian GNU/Linux 12 (bookworm)   6.6.87.2-microsoft-standard-WSL2 (amd64)   containerd://2.3.4
```

### Column Explanations:
| Column | Meaning |
| :--- | :--- |
| `NAME` | The hostname or identifier of the node (e.g., `minikube`). |
| `STATUS` | Node health (`Ready` means it is healthy and ready to accept pods). |
| `ROLES` | Role in the cluster (`control-plane` manages the cluster, worker nodes run apps). |
| `AGE` | How long the node has been running. |
| `VERSION` | Version of the Kubernetes node agent (`kubelet`). |
| `INTERNAL-IP` | The private network IP used for communication inside the cluster. |
| `EXTERNAL-IP` | Public IP if exposed to the internet (`<none>` in local environments). |
| `OS-IMAGE` | Operating system running on the node. |
| `KERNEL-VERSION` | Linux kernel version. |
| `CONTAINER-RUNTIME` | Container engine running the containers (e.g., `containerd`). |

---

## 3. `kubectl get deployments -A`

### What it does:
Shows all Deployments (declarative specifications that keep your application pods running and scalable) across **all namespaces**.

### Command:
```bash
kubectl get deployments -A
```

### Flags Used:
- `-A` (or `--all-namespaces`): Queries resources across every namespace, not just the `default` namespace.

### Sample Output:
```text
NAMESPACE     NAME             READY   UP-TO-DATE   AVAILABLE   AGE
default       static-website   2/2     2            2           3h42m
kube-system   coredns          0/2     2            0           4h5m
```

### Column Explanations:
| Column | Meaning |
| :--- | :--- |
| `NAMESPACE` | Logical partition holding the resource (`default` for your apps, `kube-system` for cluster internals). |
| `NAME` | Name of the deployment. |
| `READY` | `Current Ready Replicas / Desired Replicas` (e.g. `2/2` means both pods are running and healthy). |
| `UP-TO-DATE` | Number of pods updated to match the latest deployment definition. |
| `AVAILABLE` | Number of pods ready and accessible to serve traffic. |
| `AGE` | How long since the deployment was created. |

---

## 4. `kubectl get pods -A -o wide`

### What it does:
Lists every single Pod (the smallest deployable unit in Kubernetes) in all namespaces, along with its private IP and assigned node.

### Command:
```bash
kubectl get pods -A -o wide
```

### Flags Used:
- `-A`: Includes all namespaces (`default`, `kube-system`, etc.).
- `-o wide`: Shows additional details such as the Pod's internal IP address and which Node it is scheduled on.

### Sample Output:
```text
NAMESPACE     NAME                               READY   STATUS    RESTARTS      AGE     IP             NODE       NOMINATED NODE   READINESS GATES
default       static-website-54466495f9-726jb    1/1     Running   1 (28s ago)   3h42m   10.244.0.2     minikube   <none>           <none>
default       static-website-54466495f9-hzvsx    1/1     Running   1 (28s ago)   3h42m   10.244.0.4     minikube   <none>           <none>
kube-system   coredns-559f6c778d-r2svh           0/1     Running   1 (28s ago)   4h5m    10.244.0.5     minikube   <none>           <none>
kube-system   coredns-559f6c778d-xkmdd           0/1     Running   1 (28s ago)   4h5m    10.244.0.3     minikube   <none>           <none>
kube-system   etcd-minikube                      1/1     Running   1 (28s ago)   4h6m    192.168.49.2   minikube   <none>           <none>
kube-system   kindnet-r6xv9                      1/1     Running   1 (28s ago)   4h5m    192.168.49.2   minikube   <none>           <none>
kube-system   kube-apiserver-minikube            1/1     Running   1 (28s ago)   4h6m    192.168.49.2   minikube   <none>           <none>
kube-system   kube-controller-manager-minikube   1/1     Running   1 (28s ago)   4h6m    192.168.49.2   minikube   <none>           <none>
kube-system   kube-proxy-7l48n                   1/1     Running   1 (28s ago)   4h5m    192.168.49.2   minikube   <none>           <none>
kube-system   kube-scheduler-minikube            1/1     Running   1 (28s ago)   4h6m    192.168.49.2   minikube   <none>           <none>
kube-system   storage-provisioner                1/1     Running   1 (28s ago)   4h6m    192.168.49.2   minikube   <none>           <none>
```

### Column Explanations:
| Column | Meaning |
| :--- | :--- |
| `NAMESPACE` | The namespace isolation group of the Pod. |
| `NAME` | Unique pod name. Auto-generated hashes identify ReplicaSets and individual Pods. |
| `READY` | Number of running/ready containers inside the pod (e.g., `1/1`). |
| `STATUS` | Current state of the Pod (e.g., `Running`, `Pending`, `CrashLoopBackOff`, `Completed`). |
| `RESTARTS` | Number of times containers in this pod have restarted, and when the last restart occurred. |
| `AGE` | How long the pod has been alive. |
| `IP` | Virtual cluster IP assigned to the Pod. |
| `NODE` | Name of the worker or control-plane node hosting this Pod. |
| `NOMINATED NODE` | Used with scheduling priorities if preemption occurs (`<none>` by default). |
| `READINESS GATES` | Custom readiness conditions if defined (`<none>` by default). |

---

## Quick Reference Summary

| Command | Purpose |
| :--- | :--- |
| `kubectl cluster-info` | Verify cluster reachability and core control plane endpoints. |
| `kubectl get nodes -o wide` | View cluster nodes, OS, kernel, and container runtime details. |
| `kubectl get deployments -A` | View desired vs available workload replicas across all namespaces. |
| `kubectl get pods -A -o wide` | View all running pods, statuses, IP addresses, and assigned nodes. |
