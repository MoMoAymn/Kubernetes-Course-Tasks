# Task-1: Init Container & Sidecar

## What I did
I ran an nginx "hello" page on a 3-node minikube cluster. Each Pod has 3 containers:

- **init-done**: prints `done` and exits. The others start only after it succeeds.
- **nginx**: writes the page with the time and serves it.
- **time-updater**: updates the time on the page every 5 seconds.

## Files
- `namespace.yaml`
- `deployment.yaml` (for Step 5, change the init command to `exit 1`)

## Screenshots
<!-- Paste a screenshot under each line -->
- 3 nodes Ready:
- Deployment 3/3:
- Pods 2/2 Running:
- The page in the browser (time changes):
- Failing init: new Pod stuck, old Pods still running:
- After the fix, 3/3:

## Step 5 answers
- **Status of the new Pod:** `Init:CrashLoopBackOff`.
- **Do nginx and time-updater start?** No, because the init container must succeed first.
- **Is the page still working?** Yes. Kubernetes keeps the 3 old Pods running until the new one is ready, and it never gets ready.

## Fix
I applied the working `deployment.yaml` again, and all 3 Pods came back to Running.
