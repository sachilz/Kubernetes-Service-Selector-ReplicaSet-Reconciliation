# Kubernetes Workload Scaling & Port Forwarding Guide

A simple, beginner-friendly guide explaining how to access applications locally using port forwarding, scale deployments dynamically, and inspect modern EndpointSlices.

---

## 1. `kubectl port-forward svc/static-website-service 8080:80`

### What it does in simple terms:
Creates a secure tunnel directly from your local machine to the Kubernetes Service. It allows you to open `http://localhost:8080` in your web browser and view the website running inside the cluster.

### Command:
```bash
kubectl port-forward svc/static-website-service 8080:80
```

### Breaking down the command:
* `kubectl port-forward`: Starts the local port-forwarding proxy tunnel.
* `svc/static-website-service`: Targets the Service named `static-website-service`.
* `8080:80`: `<LocalPort>:<ServicePort>`. Local port `8080` on your computer forwards to port `80` on the Kubernetes Service.

### Sample Output:
```text
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
```

### Key Takeaways:
* **Testing without external exposure**: Great for testing applications locally during development without exposing them to the public internet or changing cloud firewall rules.
* **Keeps running**: This command stays open in the terminal while the tunnel is active. Press `Ctrl + C` to stop port-forwarding.

---

## 2. `kubectl scale deployment static-website --replicas=3`

### What it does in simple terms:
Tells Kubernetes to change the number of running pod copies from 2 up to 3.

### Command:
```bash
kubectl scale deployment static-website --replicas=3
```

### Breaking down the command:
* `kubectl scale`: Scales an existing workload up or down.
* `deployment static-website`: The Deployment controller to scale.
* `--replicas=3`: The new target number of pod instances you want running.

### Sample Output:
```text
deployment.apps/static-website scaled
```

### What happens under the hood:
1. Kubernetes updates the desired replica count in the Deployment.
2. The Deployment notifies its ReplicaSet.
3. The ReplicaSet immediately asks the scheduler to spin up 1 new Pod (`static-website-54466495f9-gzvw8`) to reach 3 total pods.

---

## 3. `kubectl get pods -l app=static-website -o wide`

### What it does in simple terms:
Lists all pods for your website to verify that the 3rd pod was created and has been assigned its own private IP address.

### Command:
```bash
kubectl get pods -l app=static-website -o wide
```

### Breaking down the command:
* `kubectl get pods`: Lists the pods.
* `-l app=static-website`: Shows only pods with label `app=static-website`.
* `-o wide`: Shows extra details like the pod `IP` and assigned `NODE`.

### Sample Output:
```text
NAME                              READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
static-website-54466495f9-726jb   1/1     Running   2          3d5h    10.244.0.2   minikube   <none>           <none>
static-website-54466495f9-gzvw8   1/1     Running   0          2m20s   10.244.0.6   minikube   <none>           <none>
static-website-54466495f9-hzvsx   1/1     Running   2          3d5h    10.244.0.3   minikube   <none>           <none>
```

### What the output means:
* You now have **3 running pods** instead of 2.
* The newly created pod (`gzvw8`, age ~2m) was automatically assigned private IP `10.244.0.6`.
* All 3 pods report `READY: 1/1` and `STATUS: Running`.

---

## 4. `kubectl get endpointslices -l kubernetes.io/service-name=static-website-service`

### What it does in simple terms:
Inspects the modern Kubernetes **EndpointSlice** to confirm that the new 3rd pod's IP was automatically registered to receive traffic from the Service.

### Command:
```bash
kubectl get endpointslices -l kubernetes.io/service-name=static-website-service
```

### Breaking down the command:
* `get endpointslices`: Queries EndpointSlices (the modern, scalable replacement for traditional Endpoints in Kubernetes).
* `-l kubernetes.io/service-name=static-website-service`: Filters EndpointSlices specifically created for `static-website-service`.

### Sample Output:
```text
NAME                           ADDRESSTYPE   PORTS   ENDPOINTS                          AGE
static-website-service-5ldpp   IPv4          80      10.244.0.2,10.244.0.3,10.244.0.6   3d5h
```

### What the output means:
* **ENDPOINTS (`10.244.0.2, 10.244.0.3, 10.244.0.6`)**: As soon as pod #3 came online, Kubernetes automatically detected it and added its IP (`10.244.0.6`) to the slice.
* Any web traffic sent to the Service or through the port-forward is now automatically split across all 3 backend pods.

---

## Quick Reference Summary

| Command | Simple Purpose |
| :--- | :--- |
| `kubectl port-forward svc/static-website-service 8080:80` | Access the cluster Service in your local web browser at `localhost:8080`. |
| `kubectl scale deployment static-website --replicas=3` | Instantly scale the deployment up or down to your desired replica count. |
| `kubectl get pods -l app=static-website -o wide` | Verify newly scaled pods, their health status, and assigned internal IPs. |
| `kubectl get endpointslices -l kubernetes.io/service-name=static-website-service` | Confirm all active pod IPs are registered in the Service's EndpointSlice. |

