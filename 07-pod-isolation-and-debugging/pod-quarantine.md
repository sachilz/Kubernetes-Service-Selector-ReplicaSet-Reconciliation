# Kubernetes Pod Quarantine & Isolation for Debugging

A simple, beginner-friendly guide demonstrating how to isolate a misbehaving pod from live traffic by changing its labels, and how the Deployment automatically spawns a replacement pod while keeping the isolated pod alive for offline inspection.

---

## What is "Pod Quarantine"?

In production, when a pod is throwing errors or running slowly, deleting it destroys all debug evidence (memory state, crash logs, and local file changes).

Instead of deleting it, DevOps engineers **change the pod's label**:
1. The **Service** immediately stops routing user traffic to it.
2. The **Deployment** notices it is down to 2 pods, so it automatically launches a brand new healthy 3rd pod.
3. The misbehaving pod stays running alone in "quarantine" so you can debug it safely.

---

## 1. `kubectl label pod static-website-54466495f9-8pqfb app=temporary-test --overwrite`

### What it does in simple terms:
Changes the pod's label from `app=static-website` to `app=temporary-test`. This detaches the pod from both the Service and the Deployment.

### Command:
```bash
kubectl label pod static-website-54466495f9-8pqfb app=temporary-test --overwrite
```

### Breaking down the command:
* `kubectl label pod`: Modifies labels on a pod.
* `static-website-54466495f9-8pqfb`: The specific pod being isolated.
* `app=temporary-test`: The new label value.
* `--overwrite`: Forces Kubernetes to replace the existing `app` label.

### Sample Output:
```text
pod/static-website-54466495f9-8pqfb labeled
```

---

## 2. `kubectl get endpointslices -l kubernetes.io/service-name=static-website-service`

### What it does in simple terms:
Verifies that the Service immediately kicked the isolated pod out of its load-balancing pool so users no longer hit it.

### Command:
```bash
kubectl get endpointslices -l kubernetes.io/service-name=static-website-service
```

### Sample Output:
```text
NAME                           ADDRESSTYPE   PORTS   ENDPOINTS                          AGE
static-website-service-5ldpp   IPv4          80      10.244.0.2,10.244.0.3,10.244.0.8   3d5h
```

### What changed:
* The isolated pod's IP (`10.244.0.7`) is **gone** from the Service endpoints.
* A new IP (`10.244.0.8`) took its place. Live traffic is 100% safe.

---

## 3. `kubectl get pods --show-labels`

### What it does in simple terms:
Displays all pods in the cluster along with their labels so you can see the isolated pod running alongside the active deployment pods.

### Command:
```bash
kubectl get pods --show-labels
```

### Sample Output:
```text
NAME                              READY   STATUS    RESTARTS   AGE     LABELS
static-website-54466495f9-726jb   1/1     Running   2          3d5h    app=static-website,pod-template-hash=54466495f9
static-website-54466495f9-8pqfb   1/1     Running   0          17m     app=temporary-test,pod-template-hash=54466495f9
static-website-54466495f9-hzvsx   1/1     Running   2          3d5h    app=static-website,pod-template-hash=54466495f9
static-website-54466495f9-plmlv   1/1     Running   0          3m28s   app=static-website,pod-template-hash=54466495f9
```

### Notice the 4 pods:
* **Active Pods (3)**: `726jb`, `hzvsx`, and brand new replacement pod `plmlv` (all have `app=static-website`).
* **Quarantined Pod (1)**: `8pqfb` has `app=temporary-test`. It is still alive, preserving all memory and logs for you to investigate!

---

## 4. `kubectl get deployment static-website`

### What it does in simple terms:
Confirms that the Deployment noticed the missing pod and automatically maintained full capacity (`3/3`).

### Command:
```bash
kubectl get deployment static-website
```

### Sample Output:
```text
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
static-website   3/3     3            3           3d5h
```

### What this proves:
Because `8pqfb` no longer matched `app=static-website`, the Deployment considered it "gone" from its pool and immediately created `plmlv`. Your website never experienced a dip in capacity.

---

## 5. `kubectl get pods -l app=static-website -o wide`

### What it does in simple terms:
Filters pods by the original label `app=static-website` to confirm which pods are officially part of the production application.

### Command:
```bash
kubectl get pods -l app=static-website -o wide
```

### Sample Output:
```text
NAME                              READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
static-website-54466495f9-726jb   1/1     Running   2          3d5h    10.244.0.2   minikube   <none>           <none>
static-website-54466495f9-hzvsx   1/1     Running   2          3d5h    10.244.0.3   minikube   <none>           <none>
static-website-54466495f9-plmlv   1/1     Running   0          4m      10.244.0.8   minikube   <none>           <none>
```

> **Result**: The isolated pod `8pqfb` does not appear in this list because its label was changed. Only the 3 healthy production pods appear.

---

## Quick Reference Summary

| Command | Why You Run It |
| :--- | :--- |
| `kubectl label pod <pod-name> app=temporary-test --overwrite` | Isolate a pod from live traffic and Deployment management. |
| `kubectl get endpointslices -l ...` | Confirm the isolated pod was removed from traffic routing. |
| `kubectl get pods --show-labels` | See the quarantined pod running safely alongside active pods. |
| `kubectl get deployment static-website` | Verify the Deployment auto-created a replacement pod (`3/3`). |
| `kubectl get pods -l app=static-website -o wide` | List only the active production pods receiving user requests. |

