# Task-1: Init Container & Sidecar

## What I did
I ran an nginx "hello" page on a 3-node minikube cluster (Calico). Each Pod has 3 containers:

- **init-done**: prints `done` and exits. The others start only after it succeeds.
- **nginx**: writes the page with the current time and serves it on port 80.
- **time-updater**: updates the time on the page every 5 seconds.

The Pods have no volumes.

## Files
- `namespace.yaml`: creates the `webapp` namespace
- `deployment.yaml`: the Deployment (3 replicas)

## Steps and screenshots

### 1. Create the cluster
`minikube start --nodes 3 --cni=calico --cpus 2 --memory 2200`, then `kubectl get nodes`:

![nodes](images/containers.jpeg)

### 2. Deploy
`kubectl apply -f namespace.yaml`, `kubectl apply -f deployment.yaml`, then `kubectl get pods -n webapp`:

![pods](images/pods.jpeg)

### 3. The page in the browser
`kubectl port-forward -n webapp deploy/web-deployment 8080:80`, then open http://localhost:8080:

![page 1](images/timer-1.jpeg)

After 5 seconds, refreshed. The time changed:

![page 2](images/timer-2.jpeg)

### 4. Make the init container fail
I changed the init command to `echo 'init failed'; exit 1` and ran `kubectl apply -f deployment.yaml`.

The new Pod is stuck in `Init:CrashLoopBackOff`, and the 3 old Pods are still running:

![failing init](images/fail-init.jpeg)

`kubectl describe pod <new-pod> -n webapp` shows `Exit Code: 1`:

![describe](images/describe-fail.jpeg)

**Answers**
- **Status of the new Pod:** `Init:CrashLoopBackOff`.
- **Do nginx and time-updater start?** No, because the init container must succeed first.
- **Is the page still working?** Yes. Kubernetes keeps the 3 old Pods running until the new one is ready, and it never gets ready.

### 5. Fix it
I changed the init command back to `echo done` and applied `deployment.yaml` again. All 3 Pods are running again:

![fixed](images/fix.jpeg)

## Verification

**Container names**

`kubectl get pods -n webapp -l app=web -o jsonpath='{.items[0].spec.initContainers[*].name} {.items[0].spec.containers[*].name}'`

![containers](images/8-container.jpeg)

**Init container succeeded**

`kubectl logs deploy/web-deployment -n webapp -c init-done`

![init log](images/init-logs.jpeg)

**Sidecar sees nginx**

`kubectl exec deploy/web-deployment -n webapp -c time-updater -- ps`

![sidecar](images/10-sidecare.jpeg)

**Time updates**

`kubectl exec deploy/web-deployment -n webapp -c nginx -- cat /usr/share/nginx/html/index.html`

![time](images/11-timer.jpeg)

**No volumes**

`kubectl get deploy web-deployment -n webapp -o jsonpath='{.spec.template.spec.volumes}'`

![volumes](images/12-spec-volume.jpeg)
