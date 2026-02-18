
---
# Session 9
# The Architecture of Kubernetes I


## Background 

### History and Timeline:
- Kubernetes was created by Google based on its internal Borg/Omega systems and donated as open-source to the newly formed Cloud Native Computing Foundation (CNCF) in 2014. 
- The first stable (v1.0) release came in 2015. 
- All of Kubernetes is licensed under Apache 2.0 and governed by the CNCF (a Linux Foundation project).

### Name and Logo:

- The name Kubernetes (pronounced koo-ber-NET-eez) comes from Greek for “helmsman” – the person who steers a ship.
- This nautical theme is reflected in the Kubernetes logo: a ship’s wheel (helm).
- You’ll also often see Kubernetes shortened to “K8s” – the 8 stands for the eight letters between “K” and “s” in the name.

***CNCF acts like a governing foundation, helping to support and grow Kubernetes with community-driven governance and resources***


---


##  The "Why" - The Declarative Model
Before learning the components, a we must understand **The Secret Sauce** of Kubernetes: **The Declarative Model.**

*   **The Old Way (Imperative):** You give a list of instructions. "Step 1: Buy wood. Step 2: Build a chair. Step 3: Paint it blue." If the chair breaks, nothing happens unless you give a new instruction.
*   **The Kubernetes Way (Declarative):** You provide a **Desired State**. "I want a blue chair here at all times."
*   **The Magic:** If someone steals the chair, Kubernetes notices it is gone and automatically builds a new one to match your "Desired State." This is called **Reconciliation**.

---

## The Physical Layout (Nodes & Planes)
Kubernetes isn't one software; it’s a cluster of computers (Nodes) working together.

### What is Kubernetes?
- Kubernetes is a container orchestrator a platform that deploys and manages containerized applications. 
- It automatically rolls out updates, scales your apps, and self-heals failures without manual intervention. 
- In practice, you package each microservice in a container, tell Kubernetes how many replicas you want, and it takes care of running them on your servers. 
- It can deploy your app, scale it up or down, restart crashed containers, and even perform rolling updates or rollbacks as needed.

###  The Node (The Computer)
A **Node** is simply a machine it can be a physical server in a closet, a Virtual Machine (VM) in the cloud, or even a Raspberry Pi. 
*   **A cluster** is a group of these machines pooled together into one "Super Computer."

### The Control Plane (The Brain)
The Control Plane makes the decisions. It doesn't run your website or your app; it manages the cluster. It is the "Command Center." If the Control Plane says "Run a web server," it finds a worker node to do it.

### The Data Plane (The Muscle)
Also known as **Worker Nodes**. These machines do the heavy lifting. They host the containers where your actual code (the website, the database) lives.

---

## Deep Dive into Control Plane Components
*Nigel Poulton calls these the "Brains of the cluster."*

###  The API Server (The Grand Central Station)
*   **Deep Explanation:** In Kubernetes, **nothing** happens without the API Server. It is the only component that speaks to the user and the only component that speaks to the database (etcd).
*   **The Logic:** When you type a command into your computer, it sends a JSON (text) message to the API Server. The API Server checks: 
    1. **Authentication:** "Who are you?" 
    2. **Authorization:** "Do you have permission to do this?"
    3. **Admission Control:** "Is this request safe for the cluster?"

### etcd (The Cluster Store)
*   **Deep Explanation:** This is the cluster's **Memory**. It stores the "Single Source of Truth." 
*   **The Logic:** It stores your "Desired State." If you want 5 copies of an app, `etcd` remembers that. It uses a system called **RAFT Consensus**, which means if you have multiple copies of `etcd`, they all agree on the data before it's saved. **If etcd is lost, your cluster is dead.**

### The Scheduler (The Logistics Expert)
*   **Deep Explanation:** The Scheduler is a "matchmaker." It looks at a new task and looks at the available Worker Nodes.
*   **The Logic:** It uses a two-step process:
    1. **Filtering:** "Which nodes actually have enough RAM to run this?"
    2. **Scoring:** "Of the nodes that fit, which one is the least busy?"
*   It then "binds" the task to that Node.

###  The Controller Manager (The Watchman)
*   **Deep Explanation:** This is a collection of "Watch Loops." It is a background process that never sleeps.
*   **The Logic:** Its only job is to compare:
    *   **Desired State** (What you asked for in etcd).
    *   **Observed State** (What is actually happening on the machines).
    *   If they don't match, the Controller Manager sends a request to the API Server to fix it.

---

## Deep Dive into Worker Node Components
*These components live on every single machine in the Data Plane.*

###  The Kubelet (The Foreman)
*   **Deep Explanation:** The Kubelet is the "Main Agent." It is the bridge between the Brain (Control Plane) and the Muscle (Worker Node).
*   **The Logic:** The Kubelet watches the API Server. When the API Server says, "Hey, the Scheduler picked your machine to run this app," the Kubelet takes the order and makes sure the container starts. It then constantly reports back: "App is still running!" or "App crashed, I'm restarting it!"

### The Container Runtime (The Engine)
*   **Deep Explanation:** Kubernetes does not actually know how to run a container. It needs a specialized engine.
*   **The Logic:** Common engines are **containerd** or **CRI-O**. The Kubelet tells the Container Runtime: "Download this image and start it up."

### Kube-proxy (The Traffic Cop)
*   **Deep Explanation:** Networking in a cluster is hard because containers move around and their IP addresses change constantly (IP Churn).
*   **The Logic:** Kube-proxy manages the "Phone Book" of the node. It ensures that if a user wants to visit your website, the traffic gets sent to the right container on the right machine, even if that container just moved.

---

