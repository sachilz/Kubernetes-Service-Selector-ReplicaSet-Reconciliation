# Kubernetes Workload & Service Inspection Commands

A simple, beginner-friendly guide explaining the key `kubectl` commands used to inspect system pods, application deployments, network services, and replica controllers.

---

## 1. `kubectl describe pods -n kube-system -l k8s-app=kube-dns`

### What it does in simple terms:
Gives you a detailed "medical report" for the internal DNS system (**CoreDNS**). It shows you if DNS is healthy, its assigned IP, container settings, and any error events.

### Command:
```bash
kubectl describe pods -n kube-system -l k8s-app=kube-dns
```

### Breaking down the command:
* `kubectl describe`: Prints a deep diagnostic report instead of just a summary line.
* `pods`: Tells Kubernetes you want to inspect Pods.
* `-n kube-system`: Looks inside the `kube-system` folder (namespace) where Kubernetes keeps its own internal background services.
* `-l k8s-app=kube-dns`: A label filter. It only shows pods labeled `k8s-app=kube-dns` so you don't have to look up their random generated pod names.

### Sample Output:
```text
Name:                 coredns-559f6c778d-r2svh
Namespace:            kube-system
Node:                 minikube/192.168.49.2
Status:               Running
IP:                   10.244.0.4
Controlled By:        ReplicaSet/coredns-559f6c778d
Containers:
  coredns:
    Image:            registry.k8s.io/coredns/coredns:v1.14.6
    State:            Running
    Ready:            True
    Restart Count:    2
Conditions:
  Type                Status
  Ready               True 
Events:               <none>
```

### What the output means:
* **Status (`Running`)**: The pod is alive and functioning.
* **Controlled By**: The parent controller managing this pod (a ReplicaSet).
* **Ready (`True`)**: The container passed its health checks and is ready to resolve DNS requests for other pods.
* **Events (`<none>`)**: No recent errors. If something breaks, warning logs will appear here at the bottom.

---

## 2. `kubectl get pods -n kube-system`

### What it does in simple terms:
Shows a quick list of all the core engine components that keep the Kubernetes cluster running.

### Command:
```bash
kubectl get pods -n kube-system
```

### Breaking down the command:
* `kubectl get`: Shows a simple summary table.
* `pods`: The resource type to display.
* `-n kube-system`: Targets the system namespace.

### Sample Output:
```text
NAME                               READY   STATUS    RESTARTS      AGE
coredns-559f6c778d-r2svh           1/1     Running   2 (21m ago)   3d5h
coredns-559f6c778d-xkmdd           1/1     Running   2 (21m ago)   3d5h
etcd-minikube                      1/1     Running   2 (21m ago)   3d5h
kindnet-r6xv9                      1/1     Running   2 (21m ago)   3d5h
kube-apiserver-minikube            1/1     Running   2 (21m ago)   3d5h
kube-controller-manager-minikube   1/1     Running   2 (21m ago)   3d5h
kube-proxy-7l48n                   1/1     Running   2 (21m ago)   3d5h
kube-scheduler-minikube            1/1     Running   2 (21m ago)   3d5h
storage-provisioner                1/1     Running   2 (21m ago)   3d5h
```

### Simple Explanation of Key Components:
* **`kube-apiserver`**: The front door/brain. It receives and processes all `kubectl` commands.
* **`etcd`**: The cluster's database. It stores the state and configuration of everything in the cluster.
* **`kube-scheduler`**: The matchmaker. It decides which node runs newly created pods based on available CPU and memory.
* **`kube-controller-manager`**: The regulator. It continuously watches the cluster to make sure actual state matches desired state.
* **`kube-proxy`**: The traffic cop. It manages networking rules on each node so traffic gets to the right pod.
* **`coredns`**: The phonebook. Allows pods to find each other using names instead of changing IP addresses.

---

## 3. `kubectl describe deployment static-website -n default`

### What it does in simple terms:
Shows the complete blueprint, scaling settings, container image, and rollout status of your `static-website` application.

### Command:
```bash
kubectl describe deployment static-website -n default
```

### Breaking down the command:
* `describe deployment`: Tells Kubernetes to show detailed properties of a Deployment.
* `static-website`: The specific name of your deployment.
* `-n default`: Looks in the standard namespace where user apps live.

### Sample Output:
```text
Name:                   static-website
Namespace:              default
Selector:               app=static-website
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
Pod Template:
  Labels:  app=static-website
  Containers:
   nginx:
    Image:      static-website:v1
    Port:       80/TCP
    Limits:
      cpu:     250m
      memory:  128Mi
    Requests:
      cpu:         50m
      memory:      32Mi
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
NewReplicaSet:   static-website-54466495f9 (2/2 replicas created)
```

### What the output means:
* **Replicas (`2 desired | 2 available`)**: You asked for 2 copies of your website, and exactly 2 are currently up and running.
* **StrategyType (`RollingUpdate`)**: When you update the image, Kubernetes swaps pods one by one with zero downtime.
* **Pod Template**: The recipe used to build pods (uses the image `static-website:v1` and listens on port 80).
* **Conditions (`Available = True`)**: Confirms the minimum required pods are running and serving traffic.

---

## 4. `kubectl get svc -A`

### What it does in simple terms:
Shows all network doors (Services) across the entire cluster. Services give your pods static IP addresses and port numbers so they can be accessed.

### Command:
```bash
kubectl get svc -A
```

### Breaking down the command:
* `get svc`: Short for `get services`.
* `-A`: Short for `--all-namespaces`. Shows services across every namespace at once.

### Sample Output:
```text
NAMESPACE     NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes               ClusterIP   10.96.0.1        <none>        443/TCP                  3d5h
default       static-website-service   NodePort    10.104.117.163   <none>        80:30080/TCP             3d5h
kube-system   kube-dns                 ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   3d5h
```

### Column Explanations:
| Column | Meaning in Simple Words |
| :--- | :--- |
| `NAMESPACE` | Which section the service belongs to (`default` or `kube-system`). |
| `NAME` | The service name. Pods use this name like a web URL to communicate. |
| `TYPE` | How it is exposed: `ClusterIP` (internal only) or `NodePort` (accessible from outside the cluster). |
| `CLUSTER-IP` | Fixed internal IP address used inside the cluster. |
| `PORT(S)` | Port numbers. For example, `80:30080/TCP` means internal port `80` is mapped to outside port `30080`. |

---

## 5. `kubectl get rs -n default`

### What it does in simple terms:
Shows your ReplicaSets. A ReplicaSet acts like a guard whose only job is to guarantee that the desired number of pods are always running.

### Command:
```bash
kubectl get rs -n default
```

### Breaking down the command:
* `get rs`: Short for `get replicasets`.
* `-n default`: Only checks the `default` namespace.

### Sample Output:
```text
NAME                        DESIRED   CURRENT   READY   AGE
static-website-54466495f9   2         2         2       3d5h
```

### Column Explanations:
| Column | Meaning in Simple Words |
| :--- | :--- |
| `NAME` | The ReplicaSet name (Deployment name + unique template hash). |
| `DESIRED` | How many pods you want running (2). |
| `CURRENT` | How many pods currently exist on the cluster (2). |
| `READY` | How many pods are fully started and ready to serve traffic (2). |

> **Analogy**: If one of your pods crashes or is deleted, the ReplicaSet immediately spots the missing pod (`READY: 1`) and creates a new replacement pod automatically until `READY: 2` is restored.

---

## Summary Cheat Sheet

| Command | Simple Purpose |
| :--- | :--- |
| `kubectl describe pods -n kube-system -l k8s-app=kube-dns` | Deep-dive health and error check for CoreDNS. |
| `kubectl get pods -n kube-system` | Quick status check of all core Kubernetes engine components. |
| `kubectl describe deployment static-website -n default` | Blueprint, replica counts, and container specs for your app. |
| `kubectl get svc -A` | List all network endpoints and exposed ports in the cluster. |
| `kubectl get rs -n default` | Check the ReplicaSet guard keeping your pod counts steady. |
