# 🔟 Kubernetes MCP Test Scenarios (DEMO)

---

## ✅ Case 1: 3-Tier Application (Frontend + Backend + DB)

### Deploy

```bash
kubectl apply -f https://k8s.io/examples/application/guestbook/guestbook-all-in-one.yaml
```

### Ask MCP

```text
Analyze this 3-tier application.
Are frontend, backend, and Redis properly connected?
Check service discovery and pod health.
```

👉 MCP validates:

* Service → Pod labels
* DNS
* Endpoints

---

## ❌ Case 2: Application Down (Service has no endpoints)

Delete backend pods:

```bash
kubectl delete pod -l app=redis
```

### Ask MCP

```text
Why is the frontend application not reachable?
Check service endpoints and root cause.
```

👉 MCP finds:

* Service exists
* No endpoints
* Backend pods missing

---

## ❌ Case 3: ImagePullBackOff

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: imagepull-demo
spec:
  containers:
  - name: app
    image: nginx:notexist
```

```bash
kubectl apply -f imagepull.yaml
```

### Ask MCP

```text
Why is this pod in ImagePullBackOff?
Check events and suggest fix.
```

👉 MCP explains:

* Image not found
* Registry / tag issue

---

## ❌ Case 4: CrashLoopBackOff

```yaml
containers:
- name: crash
  image: busybox
  command: ["sh", "-c", "exit 1"]
```

### Ask MCP

```text
Why is this pod restarting continuously?
Check logs and exit reason.
```

👉 MCP:

* Reads logs
* Identifies exit code
* Suggests fix

---

## ❌ Case 5: Pod Not Scheduling (Wrong nodeSelector)

```yaml
nodeSelector:
  disktype: ssd
```

(No node has this label)

### Ask MCP

```text
Why is this pod stuck in Pending?
Check scheduling constraints.
```

👉 MCP checks:

* nodeSelector
* node labels
* scheduler events

---

## ❌ Case 6: Insufficient CPU / Memory

```yaml
resources:
  requests:
    cpu: "10"
```

### Ask MCP

```text
Pod is Pending. Is this due to resource constraints?
Check node capacity vs requests.
```

👉 MCP:

* Compares node allocatable
* Explains scheduling failure

---

## ❌ Case 7: ConfigMap Missing

```yaml
envFrom:
- configMapRef:
    name: app-config-missing
```

### Ask MCP

```text
Why is this pod failing to start?
Check configuration dependencies.
```

👉 MCP:

* Detects missing ConfigMap
* Shows exact error

---

## ❌ Case 8: Secret Missing

```yaml
secretKeyRef:
  name: db-secret
```

### Ask MCP

```text
Application is crashing at startup.
Check if secrets are properly configured.
```

👉 MCP:

* Finds missing secret
* Explains security best practice

---

## ❌ Case 9: Liveness Probe Killing Pod

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
```

(App listens on 3000)

### Ask MCP

```text
Why is Kubernetes killing this pod repeatedly?
Analyze probes.
```

👉 MCP:

* Matches probe vs container port
* Suggests correction

---

## ❌ Case 10: Deployment Rollout Stuck

```bash
kubectl rollout status deploy/frontend
```

### Ask MCP

```text
Why is this deployment rollout stuck?
Analyze replica set and pod state.
```

👉 MCP:

* Checks old vs new ReplicaSets
* Finds failing pods
* Suggests rollback

---

## 🧠 What MCP Is Doing Internally

For each question MCP:

* Runs kubectl read-only calls
* Inspects:

  * pod status
  * events
  * logs
  * YAML diffs
* Builds **human-style RCA**

---

## 🎯 Final Outcome

After this demo you can:

* Debug Kubernetes using **natural language**
* Reduce MTTR drastically
* Use MCP as **on-call assistant**
* Demo this in interviews / team sessions

---

## 🚀 Next Level (Optional)

Want me to give:

* 🔐 Read-only RBAC for MCP?
* 🤖 Auto-RCA templates?
* 🧪 Chaos testing + MCP?
* 📝 Confluence demo doc?
* 🎥 Live demo script for presentation?

Just tell me 👍
