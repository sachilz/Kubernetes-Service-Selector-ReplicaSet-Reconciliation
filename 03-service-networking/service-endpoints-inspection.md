# Kubernetes Service Networking & Endpoints Inspection

A simple, beginner-friendly guide explaining how Kubernetes Services connect to Pods using Endpoints, port mappings, and YAML configurations.

---

## 1. `kubectl describe svc static-website-service`

### What it does in simple terms:
Gives you the complete networking profile for a Service. It shows what ports are open, the Service's internal IP, and which backend Pods (**Endpoints**) are receiving the traffic.

### Command:
```bash
kubectl describe svc static-website-service
```

### Breaking down the command:
* `kubectl describe`: Prints a detailed diagnostic view.
* `svc`: Short for `service`.
* `static-website-service`: The exact name of the Service.

### Sample Output:
```text
Name:                     static-website-service
Namespace:                default
Selector:                 app=static-website
Type:                     NodePort
IP:                       10.104.117.163
Port:                     80/TCP
TargetPort:               80/TCP
NodePort:                 30080/TCP
Endpoints:                10.244.0.2:80,10.244.0.3:80
```

### What the output means:
* **Selector (`app=static-website`)**: The Service looks for any Pod with the label `app=static-website` to route traffic to.
* **Type (`NodePort`)**: Opens a port on your Minikube node (`30080`) so you can access the website from outside the cluster.
* **Port (`80/TCP`)**: The internal port inside the cluster.
* **TargetPort (`80/TCP`)**: The port your container (Nginx) is actually listening on inside the pod.
* **Endpoints (`10.244.0.2:80, 10.244.0.3:80`)**: The real private IP addresses and ports of the live Pods currently serving traffic.

---

## 2. `kubectl get endpoints static-website-service`

### What it does in simple terms:
Shows only the list of live Pod IP addresses that are actively hooked up to the Service.

### Command:
```bash
kubectl get endpoints static-website-service
```

### Breaking down the command:
* `get endpoints`: Queries the Endpoints resource.
* `static-website-service`: The Service name to check.

### Sample Output:
```text
NAME                     ENDPOINTS                     AGE
static-website-service   10.244.0.2:80,10.244.0.3:80   3d5h
```

### What the output means:
* **ENDPOINTS**: Shows `10.244.0.2:80` and `10.244.0.3:80`. Whenever traffic hits `static-website-service`, Kubernetes automatically balances the load between these two pod IP addresses.
* **Troubleshooting Tip**: If `ENDPOINTS` is empty (`<none>`), it means either your pods aren't running, they failed readiness checks, or the Service's `selector` doesn't match the pod `labels`.

---

## 3. `kubectl get pods -l app=static-website -o wide`

### What it does in simple terms:
Lists your application Pods along with their private IP addresses and assigned nodes, confirming that they match the Service's Endpoints.

### Command:
```bash
kubectl get pods -l app=static-website -o wide
```

### Breaking down the command:
* `kubectl get pods`: Lists the pods.
* `-l app=static-website`: Filters to only display pods that have the label `app=static-website`.
* `-o wide`: Shows extra columns, including the Pod's internal cluster `IP` and which `NODE` it is running on.

### Sample Output:
```text
NAME                              READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
static-website-54466495f9-726jb   1/1     Running   2          3d5h   10.244.0.2   minikube   <none>           <none>
static-website-54466495f9-hzvsx   1/1     Running   2          3d5h   10.244.0.3   minikube   <none>           <none>
```

### How this connects to the Service:
Notice the **IP** column:
* Pod 1 has IP: `10.244.0.2`
* Pod 2 has IP: `10.244.0.3`

These exact IPs match the **Endpoints** listed in `kubectl get endpoints static-website-service` (`10.244.0.2:80,10.244.0.3:80`). This confirms your Service is properly linked to your running pods.

---

## 4. `kubectl get svc static-website-service -o yaml`

### What it does in simple terms:
Dumps the complete live YAML definition of the Service directly from the Kubernetes cluster database (`etcd`), including system metadata, generated cluster IPs, and port specifications.

### Command:
```bash
kubectl get svc static-website-service -o yaml
```

### Breaking down the command:
* `get svc static-website-service`: Fetches the service resource.
* `-o yaml`: Formats the entire resource configuration as readable YAML text instead of a single table row.

### Sample Output:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: static-website-service
  namespace: default
spec:
  clusterIP: 10.104.117.163
  ports:
  - nodePort: 30080
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: static-website
  type: NodePort
```

### Key YAML Sections Explained:
* `spec.type: NodePort`: Defines this as a NodePort service, exposing the service outside the cluster on port `30080`.
* `spec.clusterIP`: The internal virtual IP assigned by Kubernetes (`10.104.117.163`).
* `spec.ports`:
  * `port: 80`: The port used inside the cluster.
  * `targetPort: 80`: The port inside the container receiving requests.
  * `nodePort: 30080`: The public port opened on every cluster node.
* `spec.selector.app: static-website`: Tells Kubernetes to automatically discover pods with label `app=static-website` and add them as endpoints.

---

## Quick Reference Summary

| Command | Why You Run It |
| :--- | :--- |
| `kubectl describe svc static-website-service` | Full networking health check (ports, selector, type, and live endpoints). |
| `kubectl get endpoints static-website-service` | Quick check to see which pod IPs are currently receiving traffic from the service. |
| `kubectl get pods -l app=static-website -o wide` | Verify that the Pods' internal IPs match the Service's Endpoints. |
| `kubectl get svc static-website-service -o yaml` | View the exact live YAML configuration and port mappings for the Service. |

