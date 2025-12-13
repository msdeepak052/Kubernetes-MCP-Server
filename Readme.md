# **Kubernetes MCP inside VS Code Remote – WSL**

# 🧩 Prerequisites (you already have most)

* Windows 10/11
* WSL installed with Ubuntu
* `kubectl`, `aws`, `minikube` already working in WSL
* VS Code installed on Windows

---

# 🟢 STEP 1 — Install **Remote – WSL** extension

1. Open **VS Code (Windows)**
2. Go to **Extensions** (`Ctrl + Shift + X`)
3. Search for:

  ```
  WSL
   ```
4. Install it
5. Restart VS Code once

---

# 🟢 STEP 2 — Open VS Code **inside WSL**

1. Press:

   ```
   Ctrl + Shift + P
   ```
2. Run:

   ```
   Remote-WSL: New Window
   ```
3. A **new VS Code window opens**
4. Bottom-left corner must show something like:

   ```
   WSL: Ubuntu
   ```

⚠️ If you still see `><` or nothing → you are NOT in WSL.

---

# 🟢 STEP 3 — Open your project inside WSL

In the **WSL VS Code window**:

1. `File → Open Folder`
2. Choose a folder under:

   ```
   /home/deepak/
   ```

   Example:

   ```
   /home/deepak/work/k8s
   ```
3. Click **OK**
4. If prompted → “Trust the authors” → **Yes**

---

# 🟢 STEP 4 — Verify you are REALLY in WSL

Open terminal inside VS Code:

```
Terminal → New Terminal
```

Run:

```bash
whoami
pwd
```

Expected:

```
deepak
/home/deepak/...
```

Then:

```bash
kubectl config get-contexts
```

You should see:

* EKS clusters
* minikube
* docker-desktop

✅ This confirms MCP will see the same kubeconfig.

---

# 🟢 STEP 5 — (Important) Set a SAFE default context

### If you want local testing:

```bash
kubectl config use-context minikube
```

### If you want EKS:

```bash
kubectl config use-context arn:aws:eks:ap-south-1:339712902352:cluster/deepak-eks
```

Verify:

```bash
kubectl get ns
```

---

# 🟢 STEP 6 — Install Node.js inside WSL (required for MCP)

Inside the **WSL VS Code terminal**:

```bash
node -v
```

If command NOT found, install Node LTS:

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs
```

Verify:

```bash
node -v
npm -v
npx -v
```

---

# 🟢 STEP 7 — Configure MCP Kubernetes server (WSL)

Install MCP Server from below

[MCP Server](https://github.com/containers/kubernetes-mcp-server?tab=readme-ov-file)

<img width="1575" height="655" alt="image" src="https://github.com/user-attachments/assets/d7526dfd-b2f2-4268-bc04-473bb958c1cc" />

Open **Settings JSON** in **WSL VS Code**:

```
Ctrl + Shift + P
→ Preferences: Open Settings (JSON)
```
Check

```json
{
  "mcpServers": {
    "kubernetes": {
      "command": "npx",
      "args": [
        "-y",
        "@kubernetes/mcp-server",
        "--log-level",
        "4"
      ]
    }
  }
}
```

Save file.

---

# 🟢 STEP 8 — Start Kubernetes MCP Server

1. `Ctrl + Shift + P`
2. Run:

   ```
   MCP: Start Server
   ```
3. Select:

   ```
   kubernetes
   ```

---

# 🟢 STEP 9 — Confirm SUCCESS

In MCP logs you should see:

```
Connection state: Running
Waiting for server to respond to initialize
Connection state: Ready
```

❌ No `invalid configuration`
❌ No minikube cert errors
❌ No process exit

---

# 🟢 STEP 10 — Final sanity test

In VS Code chat / Make sure the Copilot is connected through agent:

<img width="686" height="167" alt="image" src="https://github.com/user-attachments/assets/dadc7cf0-b0e6-406f-a0c2-7ad9d7b4faec" />


Try:

```
List all namespaces
```

or:

```
Show nodes in the current cluster
```

You should get **real Kubernetes output**.

---

# 🔒 Optional (Highly Recommended for EKS)

### Read-only mode (safe for prod):

```json
"--read-only"
```

### Disable destructive tools:

```json
"--disable-destructive"
```

Example:

```json
"args": [
  "-y",
  "@kubernetes/mcp-server",
  "--read-only",
  "--disable-destructive"
]
```

---

# 🧠 Why this setup is CORRECT

* MCP runs where **kubectl + certs** live
* No Windows ↔ WSL filesystem hacks
* Works with:

  * EKS
  * minikube
  * multiple clusters
* This is how AI-driven platform tooling is expected to run

---

