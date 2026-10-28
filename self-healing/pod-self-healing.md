# Kubernetes Self-Healing & Pod Lifecycle Guide

A simple, beginner-friendly guide demonstrating Kubernetes self-healing: what happens when a pod fails or is deleted, how the ReplicaSet automatically creates a replacement pod, and how traffic is dynamically rerouted.

---

## 1. `kubectl get pods -l app=static-website -o wide`

### What it does in simple terms:
Takes a snapshot of all running pods, their unique names, and their private IP addresses before testing self-healing.

### Command:
```bash
kubectl get pods -l app=static-website -o wide
```

### Breaking down the command:
* `kubectl get pods`: Lists the pods.
* `-l app=static-website`: Filters only pods labeled `app=static-website`.
* `-o wide`: Shows extra details like the pod `IP` and assigned `NODE`.

### Sample Output (Before Deletion):
```text
NAME                              READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
static-website-54466495f9-726jb   1/1     Running   2          3d5h   10.244.0.2   minikube   <none>           <none>
static-website-54466495f9-gzvw8   1/1     Running   0          2m     10.244.0.6   minikube   <none>           <none>
static-website-54466495f9-hzvsx   1/1     Running   2          3d5h   10.244.0.3   minikube   <none>           <none>
```

> **Target Pod**: Notice pod `static-website-54466495f9-gzvw8` running with IP `10.244.0.6`. We will delete this pod to simulate a failure.

---

## 2. `kubectl delete pod static-website-54466495f9-gzvw8`

### What it does in simple terms:
Manually deletes a pod to simulate a crash, hardware failure, or unexpected outage.

### Command:
```bash
kubectl delete pod static-website-54466495f9-gzvw8
```

### Breaking down the command:
* `kubectl delete pod`: Tells Kubernetes to terminate a specific pod.
* `static-website-54466495f9-gzvw8`: The exact name of the pod being destroyed.

### Sample Output:
```text
pod "static-website-54466495f9-gzvw8" deleted
```

---

## 3. `kubectl get pods -l app=static-website -w`

### What it does in simple terms:
Enables **watch mode** (`-w`), which streams live, real-time updates as Kubernetes detects the missing pod and automatically creates a new replacement.

### Command:
```bash
kubectl get pods -l app=static-website -w
```

### Breaking down the command:
* `-w` (or `--watch`): Keeps the command running and prints a new line immediately whenever a pod's status changes. Press `Ctrl + C` to exit.

### Sample Output:
```text
NAME                              READY   STATUS        RESTARTS   AGE
static-website-54466495f9-gzvw8   1/1     Terminating   0          3m
static-website-54466495f9-8pqfb   0/1     Pending       0          0s
static-website-54466495f9-8pqfb   0/1     ContainerCreating   0    1s
static-website-54466495f9-8pqfb   1/1     Running             0    2s
static-website-54466495f9-gzvw8   0/1     Terminating         0    3m
```

### What happened automatically (Self-Healing):
1. **Detection**: The ReplicaSet noticed that the desired count is **3**, but only **2** pods were healthy.
2. **Auto-Recovery**: It immediately requested a brand new replacement pod: `static-website-54466495f9-8pqfb`.
3. Within 2 seconds, the new pod transitioned from `Pending` → `ContainerCreating` → `Running (1/1)`.

---

## 4. `kubectl get deployment static-website`

### What it does in simple terms:
Verifies that the Deployment has fully recovered and that all desired replicas are available again.

### Command:
```bash
kubectl get deployment static-website
```

### Sample Output:
```text
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
static-website   3/3     3            3           3d5h
```

### What the output means:
* **`READY: 3/3`**: Confirms all 3 required pods are healthy and ready to serve traffic.
* **`AVAILABLE: 3`**: Zero downtime occurred during the pod crash/replacement.

---

## 5. `kubectl get endpointslices -l kubernetes.io/service-name=static-website-service`

### What it does in simple terms:
Checks the Service's network endpoints to prove that Kubernetes removed the dead pod's IP and added the new pod's IP without any manual intervention.

### Command:
```bash
kubectl get endpointslices -l kubernetes.io/service-name=static-website-service
```

### Sample Output:
```text
NAME                           ADDRESSTYPE   PORTS   ENDPOINTS                          AGE
static-website-service-5ldpp   IPv4          80      10.244.0.2,10.244.0.3,10.244.0.7   3d5h
```

### Dynamic Routing in Action:
| Stage | Active Pod IPs in EndpointSlice |
| :--- | :--- |
| **Before deletion** | `10.244.0.2, 10.244.0.3, 10.244.0.6` |
| **After self-healing** | `10.244.0.2, 10.244.0.3, 10.244.0.7` |

* Dead pod `gzvw8` (`10.244.0.6`) was **removed** from traffic routing so users never receive errors.
* New replacement pod `8pqfb` (`10.244.0.7`) was **automatically added** as soon as it passed its readiness check.

---

## Quick Reference Summary

| Command | Simple Purpose |
| :--- | :--- |
| `kubectl get pods -l app=static-website -o wide` | List all pods with their IPs before testing failure. |
| `kubectl delete pod <pod-name>` | Delete a pod to test cluster auto-recovery. |
| `kubectl get pods -l app=static-website -w` | Stream live real-time pod creation and termination events. |
| `kubectl get deployment static-website` | Confirm deployment health returned to desired count (`3/3`). |
| `kubectl get endpointslices -l kubernetes.io/service-name=...` | Verify dead pod IP was replaced with the new pod IP. |

