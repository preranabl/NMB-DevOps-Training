
# Course Title: Mastering Kubernetes: Declarative Configuration and Workload Controllers
---

## 1. The Declarative Model and Kubernetes Manifests (YAML)

Before diving into the code, it is vital to understand *how* Kubernetes thinks. Kubernetes operates on a **Declarative Model**. 

### 1.1 Imperative vs. Declarative
*   **Imperative (The Old Way):** You write long scripts of platform-specific commands to build and monitor things. It is a list of *instructions* on *how* to do something (e.g., "Start a container, check if it's running, if it fails, restart it"). 
*   **Declarative (The Kubernetes Way):** You describe the **desired state** an end goal of what you want (e.g., "I want 5 instances of this web server running"). You leave the "how" entirely up to Kubernetes.

If the *observed state* (what is actually running) varies from the *desired state* (what you asked for), a background reconciliation loop in Kubernetes will automatically attempt to fix it. 

We declare this "desired state" using **Manifest Files** written in YAML. Manifests act as excellent sources of documentation, bridging the gap between developers and operations (often called "Empathy as Code").

### 1.2  Kubernetes Manifest Strucutre 
Every Kubernetes manifest file requires four top-level fields (resources). If any of these are missing, Kubernetes will reject the file.

1.  `apiVersion`
2.  `kind`
3.  `metadata`
4.  `spec`

Let's look at a concrete example of a Pod manifest and break down every single line:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod
  labels:
    zone: prod
    version: v1
spec:
  containers:
  - name: hello-ctr
    image: my-app-image:1.0
    ports:
    - containerPort: 8080
```

#### Detailed Breakdown of the Structure:

**1. `apiVersion`**
This tells Kubernetes which schema version to use when creating the object. The Kubernetes API is massive, so it is divided into "groups" to make it manageable. 
*   The normal format is `<api-group>/<version>` (e.g., `apps/v1` for Deployments).
*   However, core objects that have been around since the very beginning of Kubernetes (like Pods and Services) live in the "core API group". The core group omits the API group name, so we just write `v1`.

**2. `kind`**
This tells Kubernetes the *type* of object you are defining. In this example, we are creating a `Pod` (the atomic, smallest unit of scheduling in Kubernetes). Other examples include `Deployment`, `Service`, `Secret`, `ConfigMap`, etc. *Note: This is case-sensitive.*

**3. `metadata`**
This section is where you attach data that helps identify the object within the cluster.
*   **`name`**: A mandatory string that uniquely identifies the object in its namespace. It must be a valid DNS name (alphanumerics, dashes, and dots).
*   **`labels`**: Extremely important key-value pairs (e.g., `zone: prod`). Labels are how Kubernetes loosely couples objects together. Later, we will see how Deployments use "Label Selectors" to find and manage specific Pods based on these exact labels.

**4. `spec`**
This is where the magic happens. The `spec` (specification) block is where you define the **desired state** of the object. The contents of `spec` are entirely dependent on the `kind` of object you are creating.
*   In our `Pod` example, the `spec` requires a `containers` list.
*   **`name`**: The name of the container running inside the Pod.
*   **`image`**: The exact container image (and tag/version) Kubernetes needs to pull from a registry to run this application.
*   **`ports`**: The network ports the container will listen on.

---

##  2. Workload Controllers - Deployments vs. ReplicaSets

Now that we know how to write a basic Pod manifest, we face a problem. Pods are mortal. If a node fails, the Pod dies, and it is gone forever. Static Pods do not self-heal, they do not scale, and they cannot be easily updated.

To get "Cloud Native" superpowers like self-healing, horizontal scaling, and zero-downtime updates, we must wrap our Pods in higher-level workload controllers. The two most important to understand are **ReplicaSets** and **Deployments**.

### 2.1 What is a ReplicaSet?
A ReplicaSet is a controller whose sole purpose is to ensure that a specified number of identical Pod replicas are running at any given time.

*   **Self-Healing:** If you request 10 replicas, and a worker node crashes taking down 2 of your Pods, the ReplicaSet controller notices that the *observed state* (8) does not match the *desired state* (10). It immediately creates 2 brand new Pods to replace them.
*   **Scalability:** If you need to handle more web traffic, you simply change your desired state from 10 to 50. The ReplicaSet brings up 40 more Pods. 

**The Limitation of ReplicaSets:**
ReplicaSets are great for keeping a specific version of an app running. However, they are *terrible* at updates. If you want to update your application from version 1.0 to version 2.0, a ReplicaSet cannot gracefully transition the traffic. 

### 2.2 What is a Deployment?
A Deployment is a higher-level controller that sits on top of, and manages, ReplicaSets. 

*   **Deployments manage ReplicaSets; ReplicaSets manage Pods.**
*   When you create a Deployment, it automatically creates a ReplicaSet in the background.

**Why use Deployments?**
Deployments bring **Rollouts** (zero-downtime rolling updates) and **Rollbacks** to the table. Modern microservices must be updated seamlessly without clients experiencing downtime.

*   **Rollouts (Updates):** When you update the image version in a Deployment's YAML file and apply it, the Deployment creates a *second*, brand-new ReplicaSet for the new version. It then systematically scales up the new ReplicaSet while simultaneously scaling down the old ReplicaSet. The result is a smooth, incremental rollout with zero downtime.
*   **Rollbacks:** Deployments do not delete the old ReplicaSets. They wind them down to 0 replicas but keep their configurations intact. If you push a bad update, the Deployment can instantly "rollback" by scaling the old ReplicaSet back up and winding the new, broken one down to 0.

### 2.3 The Deployment Manifest Structure
Let's look at the YAML code for a Deployment to see how it encapsulates the Pod template and the scaling rules.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-deploy
spec:
  replicas: 10
  selector:
    matchLabels:
      app: hello-world
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template: 
    # ---- BEGIN POD TEMPLATE ----
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: hello-pod
        image: my-app-image:1.0
        ports:
        - containerPort: 8080
    # ---- END POD TEMPLATE ----
```

#### Detailed Breakdown of the Deployment Structure:

Notice that there are multiple layers of nesting here. 

**The Deployment Level:**
*   `apiVersion: apps/v1`: Deployments live in the `apps` API group.
*   `spec.replicas`: We are asking the underlying ReplicaSet to maintain exactly 10 Pods.
*   `spec.selector.matchLabels`: This is crucial. This tells the Deployment *how to find* the Pods it is supposed to manage. It looks for any Pods in the cluster that have the label `app: hello-world`.

**The Strategy Level (`spec.strategy`):**
This block controls how the zero-downtime update will happen.
*   `maxSurge: 1`: During an update, the Deployment is allowed to create a maximum of 1 extra Pod above the desired 10 (total 11).
*   `maxUnavailable: 1`: During an update, the Deployment is allowed to let exactly 1 Pod go offline, meaning we will never have fewer than 9 running Pods.
*(This combination means the Deployment will update the Pods incrementally, ensuring continuous availability).*

**The Pod Template Level (`spec.template`):**
Everything nested under `template` is literally just a Pod manifest (minus the `apiVersion` and `kind`, which are implied). 
*   When the Deployment needs to create a new Pod, it stamps out a copy of this exact template.
*   **Notice the Labels:** The `template.metadata.labels` sets `app: hello-world`. This must perfectly match the `spec.selector.matchLabels` defined above it, otherwise, the Deployment will create Pods but won't recognize them as its own!

