
# Hands-on Lab: Creating Pods, Deployments, and Scaling


## Part 1: Working with a Static Pod

In this first exercise, we will create a single, static Pod using a declarative YAML manifest. We call this a "static" Pod because it is not managed by a higher-level controller (like a Deployment); if it dies, it stays dead.

### Step 1: Create the Pod Manifest
Create a new file named `pod.yaml` in your terminal or code editor, and paste the following configuration into it:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-first-pod
  labels:
    env: dev
    app: web
spec:
  containers:
  - name: web-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

**What is happening here?**
* We are declaring a `Pod` API object using the core `v1` API group.
* We gave it the name `my-first-pod` and attached two labels (`env: dev` and `app: web`).
* Inside the `spec`, we are telling Kubernetes to start a single container named `web-container` using the official `nginx:latest` web server image, and to expose port `80`.

### Step 2: Deploy the Pod to the Cluster
To submit this "desired state" to the Kubernetes API server, use the `kubectl apply` command.

```bash
kubectl apply -f pod.yaml
```
*Expected Output:*
> `pod/my-first-pod created`

### Step 3: Inspect the Running Pod
Now, let's ask the Kubernetes API server for the status of our Pod.

```bash
kubectl get pods
```
*Expected Output:*
> `NAME           READY   STATUS              RESTARTS   AGE`
> `my-first-pod   0/1     ContainerCreating   0          5s`

*(Note: If the status says `ContainerCreating`, Kubernetes is currently downloading the image from the registry. Run the command again after a few seconds, and it should transition to `Running`).*

For much deeper introspection, we can use the `describe` command. This will show us the IP address assigned to the Pod, the node it is running on, and a log of lifecycle events.

```bash
kubectl describe pod my-first-pod
```
Scroll to the very bottom of the output to the **Events** section. You will see a chronological list of actions Kubernetes took: scheduling the pod, pulling the image, and starting the container.

### Step 4: Clean up the Static Pod
Because this is a static Pod without a controller, if we delete it, it will not self-heal. Let's delete it before moving on to Deployments.

```bash
kubectl delete pod my-first-pod
```
*Expected Output:*
> `pod "my-first-pod" deleted`

---

## Part 2: Working with Deployments

Now we will do things the proper "Kubernetes Way." We will wrap a Pod template inside a Deployment. This will give our application the ability to self-heal and scale.

### Step 1: Create the Deployment Manifest
Create a new file named `deploy.yaml` and paste the following code into it:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-front-end
  template:
    metadata:
      labels:
        app: web-front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.23
        ports:
        - containerPort: 80
```

**What is happening here?**
* `kind: Deployment`: We are no longer creating a raw Pod. We are creating a Deployment.
* `replicas: 3`: We are asking Kubernetes to ensure that exactly 3 copies of our Pod are running at all times.
* `template`: Notice that everything under this block is just the Pod definition. 
* `selector`: The Deployment will monitor and manage any Pods that carry the label `app: web-front-end`.

### Step 2: Deploy the Application
Submit the file to the cluster:

```bash
kubectl apply -f deploy.yaml
```
*Expected Output:*
> `deployment.apps/web-deploy created`

### Step 3: Verify the Deployment and ReplicaSet
Let's look at the Deployment object:

```bash
kubectl get deploy web-deploy
```
*Expected Output:*
> `NAME         READY   UP-TO-DATE   AVAILABLE   AGE`
> `web-deploy   3/3     3            3           15s`

Behind the scenes, the Deployment automatically created a **ReplicaSet** to manage the Pods. Let's verify that:

```bash
kubectl get replicasets
```
*Expected Output:*
> `NAME                    DESIRED   CURRENT   READY   AGE`
> `web-deploy-67959776b5   3         3         3       30s`
*(Notice the alphanumeric hash `67959776b5` appended to the name. Kubernetes generates this based on your Pod template).*

Finally, let's look at the actual Pods. You should see 3 of them running:

```bash
kubectl get pods
```
*Expected Output:*
> `NAME                          READY   STATUS    RESTARTS   AGE`
> `web-deploy-67959776b5-abc12   1/1     Running   0          45s`
> `web-deploy-67959776b5-xyz34   1/1     Running   0          45s`
> `web-deploy-67959776b5-qwe56   1/1     Running   0          45s`

---

## Part 3: Scaling the Deployment

What happens if our web application suddenly gets a massive spike in traffic? We need to scale out (horizontal scaling). Kubernetes offers two ways to do this: **Imperatively** (via the command line) and **Declaratively** (via the YAML file). 

### Method A: The Imperative Way (Quick but not recommended)
You can command Kubernetes directly from the terminal to change the scale. Let's scale up to 5 replicas.

```bash
kubectl scale deploy web-deploy --replicas=5
```
*Expected Output:*
> `deployment.apps/web-deploy scaled`

Run `kubectl get pods` immediately. You will see 2 brand new Pods spinning up to join the original 3.

**The Problem with the Imperative Way:**
You successfully scaled to 5 replicas. However, your `deploy.yaml` file on your laptop still says `replicas: 3`. Your live environment is now *out of sync* with your configuration code (this is called "Configuration Drift"). If a colleague applies the YAML file later, it will scale your app back down to 3, causing an unexpected outage!

### Method B: The Declarative Way (The Best Practice)
To fix this, we should always make changes to the YAML file first.

1. Open `deploy.yaml` in your code editor.
2. Change the replica count from `3` to `10`:
   ```yaml
   spec:
     replicas: 10
   ```
3. Save the file.
4. "Apply" the file to the cluster again:

```bash
kubectl apply -f deploy.yaml
```
*Expected Output:*
> `deployment.apps/web-deploy configured`
*(Notice it says "configured" instead of "created" because the object already exists).*

Check your Pods again:

```bash
kubectl get pods
```
You will now see 10 Pods running happily. You just commanded a massive orchestration operation simply by changing a single number in a text file! 

### Step 4: Test Self-Healing
Because these Pods are managed by a ReplicaSet (via the Deployment), they are immortal. Let's prove it by forcefully deleting a Pod.

1. Copy the name of *one* of your running pods from the `kubectl get pods` list.
2. Delete it manually:
   ```bash
   kubectl delete pod <paste-pod-name-here>
   ```
3. Immediately run `kubectl get pods` again.

You will notice that Kubernetes instantly noticed the *observed state* (9 Pods) dropped below the *desired state* (10 Pods) and has already spun up a brand-new replacement Pod to take its place!

---

## Part 4: Lab Cleanup

To avoid leaving running resources on your cluster, always clean up using the same YAML files you used to create the resources.

```bash
kubectl delete -f deploy.yaml
```
*Expected Output:*
> `deployment.apps "web-deploy" deleted`

Run `kubectl get pods` one last time to ensure everything is terminating gracefully. You have successfully completed the core workflow of Kubernetes application management!