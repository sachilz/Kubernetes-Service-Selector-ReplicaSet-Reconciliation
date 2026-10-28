# Kubernetes Service Discovery & Label Selector Troubleshooting

A simple, beginner-friendly guide explaining how to debug Service-to-Pod connectivity, verify label-selector matching, inspect EndpointSlice YAML conditions, and extract fields using `jsonpath`.

---

## 1. `kubectl describe svc static-website-service`

### What it does in simple terms:
Checks the health and configuration of the Service to confirm that it is finding backend pods.

### Command:
```bash
kubectl describe svc static-website-service
```

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
Endpoints:                10.244.0.2:80,10.244.0.3:80,10.244.0.7:80
```

### What to check when troubleshooting:
* **Selector**: Tells you what label the Service is searching for (`app=static-website`).
* **Endpoints**: If this shows IPs, the Service is successfully linked to pods. If it shows `<none>`, there is a label mismatch or your pods are not ready.

---

## 2. `kubectl get pods -l app=static-website --show-labels`

### What it does in simple terms:
Displays all pods along with their assigned **Labels** column so you can verify that the pod labels match the Service selector character-for-character.

### Command:
```bash
kubectl get pods -l app=static-website --show-labels
```

### Breaking down the command:
* `get pods`: Lists the pods.
* `-l app=static-website`: Filters pods by label.
* `--show-labels`: Prints a dedicated `LABELS` column on the right side of the output.

### Sample Output:
```text
NAME                              READY   STATUS    RESTARTS   AGE    LABELS
static-website-54466495f9-726jb   1/1     Running   2          3d5h   app=static-website,pod-template-hash=54466495f9
static-website-54466495f9-8pqfb   1/1     Running   0          11m    app=static-website,pod-template-hash=54466495f9
static-website-54466495f9-hzvsx   1/1     Running   2          3d5h   app=static-website,pod-template-hash=54466495f9
```

### The "Golden Rule" of Service Discovery:
Every pod has `app=static-website` in its labels. Because the Service has `Selector: app=static-website`, Kubernetes connects them automatically. If the label had a typo (e.g., `app=staticwebsite`), the Service would fail to route traffic.

---

## 3. `kubectl get endpointslices -l kubernetes.io/service-name=static-website-service -o yaml`

### What it does in simple terms:
Dumps the complete YAML specification of the **EndpointSlice**. It reveals the exact health conditions (`ready: true`, `serving: true`) and which specific Pod each IP belongs to.

### Command:
```bash
kubectl get endpointslices -l kubernetes.io/service-name=static-website-service -o yaml
```

### Breaking down the command:
* `get endpointslices`: Retrieves modern EndpointSlice resources.
* `-l kubernetes.io/service-name=static-website-service`: Finds the slice belonging to this specific service.
* `-o yaml`: Formats the full internal Kubernetes object into readable YAML.

### Sample Output:
```yaml
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: static-website-service-5ldpp
  namespace: default
endpoints:
- addresses:
  - 10.244.0.2
  conditions:
    ready: true
    serving: true
    terminating: false
  targetRef:
    kind: Pod
    name: static-website-54466495f9-726jb
- addresses:
  - 10.244.0.7
  conditions:
    ready: true
    serving: true
    terminating: false
  targetRef:
    kind: Pod
    name: static-website-54466495f9-8pqfb
ports:
- port: 80
  protocol: TCP
```

### Key YAML Sections Explained:
* `addresses`: The private cluster IP of the pod receiving requests.
* `conditions.ready: true`: Confirms the container passed its readiness probe and is healthy enough to receive traffic.
* `conditions.terminating: false`: Ensures shutting-down pods are never sent traffic.
* `targetRef.name`: Directly shows the exact Pod name linked to this IP.

---

## 4. `kubectl get svc static-website-service -o jsonpath='{.spec.selector}'`

### What it does in simple terms:
Instantly extracts **only the selector key-value pairs** from the Service without printing the rest of the massive YAML file.

### Command:
```bash
kubectl get svc static-website-service -o jsonpath='{.spec.selector}'
```

### Breaking down the command:
* `get svc static-website-service`: Fetches the service definition.
* `-o jsonpath='{.spec.selector}'`: Filters the response directly at the command line to only return the `selector` field inside `spec`.

### Sample Output:
```json
{"app":"static-website"}
```

### Why use `jsonpath`:
* **Scripting and Automation**: Easily capture configuration values inside bash scripts or CI/CD pipelines.
* **Instant verification**: No need to scroll through 40 lines of YAML just to see what labels the service expects.

---

## Quick Reference Summary

| Command | Simple Purpose |
| :--- | :--- |
| `kubectl describe svc static-website-service` | Verify if the Service found backend Pods (`Endpoints`). |
| `kubectl get pods -l app=static-website --show-labels` | View all labels attached to each pod to check for typos. |
| `kubectl get endpointslices -l ... -o yaml` | Inspect pod health flags (`ready: true`) and target Pod references. |
| `kubectl get svc static-website-service -o jsonpath='{.spec.selector}'` | Quickly extract the exact label selector rule as JSON. |

