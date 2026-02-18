
---
# Session 10
#  Kubernetes Architecture 2

## 1. Pods vs. Containers: The Atomic Unit Logic
To understand Kubernetes, you must understand that it **never** runs a container by itself. It always encapsulates one or more containers into a **Pod**.

### Why not just use containers?
A container is like a single isolated process. However, real-world applications often need "helper" processes (logging, caching, security proxies). If these were in separate containers on different machines, they couldn't talk to each other efficiently.

### Shared Namespaces (The "Welding" Process)
When Kubernetes creates a Pod, it sets up a set of shared **Linux Namespaces** that all containers in that Pod inhabit:
*   **Network Namespace:** All containers in a Pod share the **same IP address** and the same port space. If Container A is a web server on port 80, Container B can reach it at `http://localhost:80`.
*   **UTS Namespace:** They share the same hostname.
*   **IPC Namespace:** They can talk to each other via shared memory.
*   **The "Pause" Container:** Behind the scenes, K8s runs a tiny, invisible container called `pause`. Its only job is to "hold" the Network Namespace open. If your app container crashes, the `pause` container keeps the IP address alive so that when the app restarts, its networking environment is exactly as it was.

---

## 2. DNS & kube-proxy: The Discovery Logic
In a cluster, Pods are "ephemeral" (they are born and die frequently). You can never rely on a Pod's IP address.

### The Cluster DNS (CoreDNS)
Kubernetes runs a built-in DNS service (usually **CoreDNS**). 
*   **Registration:** Every time a **Service** is created, CoreDNS automatically creates a record: `my-svc.my-namespace.svc.cluster.local`.
*   **Resolution:** When your code tries to connect to `my-svc`, the internal DNS translates that name into a **Virtual IP (ClusterIP)**. This IP is permanent, even if the Pods behind it change.

### kube-proxy (The Rule-Maker)
`kube-proxy` is a network service that runs on **every node**. It is not a traditional "proxy" that data flows through; it is a **Manager of Rules**.
*   **How it works:** It watches the API Server for changes to Services and Pods.
*   **The Modes:**
    *   **iptables mode (Default):** `kube-proxy` writes complex firewall rules into the Linux kernel. When traffic hits the node, the kernel "traps" the packet and redirects it to the correct Pod.
    *   **IPVS mode:** Used for high-performance clusters. It uses the Linux Virtual Server logic to handle load balancing much faster than iptables.
*   **Logic:** `kube-proxy` ensures that when you hit a Service IP, your request is load-balanced across all healthy Pods in that group.

---

## 3. Scheduler Internals: The Decision Logic
The Scheduler is a high-frequency loop that searches for "Unbound Pods" (Pods that exist in the system but have no assigned Node).

### The Scheduling Pipeline
1.  **Filtering (Predicates):** The Scheduler ignores nodes that don't meet the requirements.
    *   *Hard Constraints:* Does the node have enough CPU/RAM?
    *   *Node Selector:* Did the user specify a label (e.g., `disk=ssd`)?
    *   *Taints:* Is the node "reserved" for specific tasks?
2.  **Scoring (Priorities):** From the surviving nodes, it ranks them.
    *   *Least Requested:* Prefers nodes with more free resources to avoid "hotspots."
    *   *Image Locality:* Prefers nodes that already have the container image downloaded (to save time).
    *   *Affinity:* Prefers nodes where "buddy" Pods are already running.
3.  **Binding:** The Scheduler writes the name of the winning Node into the Pod object. **Crucial point:** The Scheduler does *not* start the Pod; it just tells the API Server where it *should* go. The **Kubelet** on that node sees the update and pulls the trigger.

---

## 4. Container Lifecycle: The Survival Logic
Kubernetes manages the "State" of a container from start to end

### Pod Phases
*   **Pending:** The API has it, but it’s waiting on the Scheduler or the Image download.
*   **Running:** At least one container is alive.
*   **Succeeded/Failed:** The Pod has finished its task.

### The Three Health Probes (The "Cluster Doctor")
1.  **Startup Probe:** Disables the other probes until the app is finished starting up. Perfect for heavy apps (like Java) that take 60 seconds to boot.
2.  **Liveness Probe:** Checks if the app is **alive**. If the app hits a "deadlock" or freezes, this probe fails, and K8s **kills and restarts** the container.
3.  **Readiness Probe:** Checks if the app is **ready to serve traffic**. If your app is still loading a 2GB database into memory, this probe fails, and the Service **stops sending users** to that Pod until it's ready.

### Graceful Termination
When you delete a Pod:
1.  K8s sends a **SIGTERM** signal to the app (the "Please wrap up" notice).
2.  K8s waits for a **TerminationGracePeriod** (Default 30 seconds).
3.  If the app is still running, K8s sends **SIGKILL** (The "Hard Stop").

---

# 🧪 Lab Portion: Live Architecture Walkthrough (Deep Dive)

### Task 1: Prove the Shared Network Namespace
1.  **Action:** Create a Pod with two containers: `nginx` and `alpine`.
2.  **Command:** `kubectl exec -it my-pod -c alpine -- apk add curl && curl localhost:80`
3.  **The Goal:** Observe that the `alpine` container can see the `nginx` website on its own `localhost`.
4.  **In-depth check:** Run `ip addr` in both containers. Notice the IP is identical.

### Task 2: Intercepting the "Phonebook" (DNS)
1.  **Action:** Launch a temporary "troubleshoot" Pod.
2.  **Command:** `kubectl run dns-test --image=busybox:1.28 -it --rm -- sh`
3.  **Inside the Pod:** Run `nslookup kubernetes.default`.
4.  **The Goal:** See the ClusterIP return. 
5.  **Logic Check:** Run `cat /etc/resolv.conf`. Students will see the `nameserver` points to the CoreDNS Service IP.

### Task 3: Simulating a Liveness Failure
1.  **Action:** Create a Pod that runs a script: `touch /tmp/healthy; sleep 30; rm /tmp/healthy; sleep 600`.
2.  **Add Probe:** Set a Liveness probe to check if `/tmp/healthy` exists.
3.  **Observation:** Run `kubectl get pods -w`.
4.  **The Goal:** Watch the Pod status. After 30 seconds, the file is deleted. The probe fails. K8s will show the Pod is "Running" but the **RESTARTS** column will flip from 0 to 1.
5.  **In-depth:** Run `kubectl describe pod`. Look at the **Events** section to see: *"Liveness probe failed: killing container."*

### Task 4: Tracing the Scheduler's "Hard No"
1.  **Action:** Label a node: `kubectl label nodes <your-node> disk=spinning`.
2.  **Action:** Deploy a Pod with a `nodeSelector` looking for `disk=ssd`.
3.  **The Goal:** The Pod will stay `Pending`. 
4.  **Verification:** `kubectl describe pod`. Look for the message: *"0/1 nodes are available: 1 node(s) didn't match Pod's node affinity/selector."*