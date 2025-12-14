# 🔟 Kubernetes MCP Test Scenarios (DEMO)

---

## ✅ Case 1: 3-Tier Application (Frontend + Backend + DB)

### Ask MCP to Deploy any 3 tier application

```bash
> Deploy a sample three-tier banking application (login page) in the banking namespace. Ensure the namespace is created if it does not already exist before deployment.
```

<img width="742" height="655" alt="image" src="https://github.com/user-attachments/assets/5fedb674-3e23-443a-896d-fbe371595385" />
<img width="712" height="947" alt="image" src="https://github.com/user-attachments/assets/195c103c-0253-404e-a706-23b4844d097e" />

<img width="1917" height="1025" alt="image" src="https://github.com/user-attachments/assets/15e6c1c1-9a20-4890-8ad3-5e97e59fb8b3" />


### Ask MCP

```text
Analyze this 3-tier application.
Are frontend, backend, and DB properly connected?
Check service discovery and pod health.
```
<img width="1918" height="1028" alt="image" src="https://github.com/user-attachments/assets/ec21c1a2-0fff-431d-8fea-28d088a4d55a" />

👉 MCP validates:

* Service → Pod labels
* DNS
* Endpoints

---

## ❌ Case 2: Application Down (Service has no endpoints)

Scaled down backend pods:

```bash
kubectl scale deploy backend --replicas 0 -n banking
```

### Ask MCP

```text
Why is the backend application not reachable?
Check service endpoints and root cause.
```

<img width="1934" height="1030" alt="image" src="https://github.com/user-attachments/assets/021978d1-cd61-4400-9ea6-e1f8e529da66" />


👉 MCP finds:

* Service exists
* No endpoints
* Backend pods missing
<img width="1256" height="656" alt="image" src="https://github.com/user-attachments/assets/375d2e94-8fdc-45c8-b29d-34a60e55d815" />

### Fix
<img width="1918" height="1002" alt="image" src="https://github.com/user-attachments/assets/851ac068-6d62-4343-ad3b-52a8cf87d96f" />


<img width="790" height="465" alt="image" src="https://github.com/user-attachments/assets/eb1f76c1-187a-40a8-8905-0950e9674c3a" />

### Deleting the ns banking

<img width="1918" height="1021" alt="image" src="https://github.com/user-attachments/assets/94ec4e98-803e-446f-9f3f-dfa4830f1b2d" />

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

<img width="1133" height="220" alt="image" src="https://github.com/user-attachments/assets/29497a2a-84fc-45af-b652-bcdbf5198a62" />


### Ask MCP

```text
Why is the pod in default ns is resulting in ImagePullBackOff?
Check events and suggest fix.
```
<img width="1918" height="1002" alt="image" src="https://github.com/user-attachments/assets/dd019e9b-e823-458f-a4e2-df20d90bc028" />

👉 MCP explains:

* Image not found
* Registry / tag issue

<img width="712" height="382" alt="image" src="https://github.com/user-attachments/assets/ac4b21b4-7183-4dfa-b1de-fc24cbeeeeca" />

<img width="1918" height="993" alt="image" src="https://github.com/user-attachments/assets/c8c9b1f0-b5a6-4a0f-95c9-226acb9dc11f" />

---

## ❌ Case 4: CrashLoopBackOff

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: crashloop-demo
  labels:
    app: crashloop
spec:
  restartPolicy: Always
  containers:
    - name: crash
      image: busybox:1.36
      command:
        - sh
        - -c
        - exit 1

```

### Ask MCP

```text
Why is the pod in default ns is restarting continuously?
Check logs and exit reason.
```

👉 MCP:

* Reads logs
* Identifies exit code
* Suggests fix
<img width="1913" height="1010" alt="image" src="https://github.com/user-attachments/assets/d28f0f9f-42b6-48a9-8cf7-5139e3c1ff29" />
<img width="1917" height="1027" alt="image" src="https://github.com/user-attachments/assets/495af6b2-fb05-4b4b-bbbc-a89395338c18" />


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
