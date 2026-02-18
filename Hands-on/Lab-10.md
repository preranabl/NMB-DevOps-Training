
---
# Lab 10
# Basic kubectl queries: nodes, version
---

## Part 1: The Version Query
Before running commands, you must ensure your "remote control" (`kubectl`) is compatible.
### 1. Basic Version Check
Run the following command:
```bash
kubectl version --short
```
*(Note: In newer versions of kubectl, `--short` is deprecated; if it fails, just use `kubectl version`)*.

**What you are looking at:**
*   **Client Version:** The version of `kubectl` installed on your laptop/terminal.
*   **Server Version:** The version of Kubernetes running on the **Control Plane**.
---

## Part 2: Checking Cluster Health
You want to make sure the **API Server** (Grand Central Station) is alive and talking back to you.

### 1. Cluster Info
Run:
```bash
kubectl cluster-info
```
**The Logic:** This command identifies where the API Server is running (usually an HTTPS URL) and where core services like `CoreDNS` are located. If you see a timeout error here, your `kubeconfig` is wrong or your cluster is down.

---

## Part 3: Investigating Nodes 
The Nodes are the machines in the **Data Plane** that do the actual work.

### 1. List all Nodes
Run:
```bash
kubectl get nodes
```
**Column Breakdown:**
*   **NAME:** The hostname of the machine.
*   **STATUS:** Should be `Ready`. If it says `NotReady`, the **Kubelet** on that machine is likely having issues.
*   **ROLES:** Tells you if the node is a `control-plane` node or a worker node.
*   **VERSION:** The version of the **Kubelet** running on that specific node.

### 2. The "Wide" Query
Sometimes the basic table isn't enough. We want to see IP addresses and the OS.
Run:
```bash
kubectl get nodes -o wide
```
**New Information Gained:**
*   **INTERNAL-IP:** The private IP address the nodes use to talk to each other.
*   **OS-IMAGE:** Are you running Ubuntu, Amazon Linux, or something else?
*   **CONTAINER-RUNTIME:** This tells you the "Engine" (e.g., `containerd://1.6.0`).

---

## Part 4: The Deep Dive (Describe)
`get` gives you a summary. `describe` gives you a full medical report of the node.

### 1. Inspecting a specific Node
Run:
```bash
# Replace <node-name> with a name from Task 3.1
kubectl describe node <node-name>
```

**Find these items in the output:**
1.  **Conditions:** Look for `OutOfDisk`, `MemoryPressure`, or `DiskPressure`. If these are `True`, the node is struggling!
2.  **Capacity vs. Allocatable:** How much CPU/RAM does the machine have vs. how much is left for your apps?
3.  **Events:** This is a log at the bottom. It shows if the node recently joined the cluster or if the Kubelet crashed.

---

## Part 5: Formatting for Automation
If you were a developer writing a script, you wouldn't want a table; you'd want raw data.

### 1. Exporting to JSON/YAML
Run:
```bash
kubectl get nodes -o json
# OR
kubectl get nodes -o yaml
```
**The Logic:** This shows you exactly how the node data is stored in **etcd**. Every detail about that machine is represented in this text format.

---
