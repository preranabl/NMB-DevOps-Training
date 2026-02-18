# **Lab 9** 
# **Architecture  walkthrough**
# **Hands-On Labs**

<!-- notes: Everyone should have `kubectl` configured and access to a cluster.  
We will use a simple payment-demo app (nginx-based) for realism.  
Allocate ~12–15 mins per exercise. Encourage typing along. -->

## **0. SSH into the Master Node**
```bash
ssh <username>@10.250.3.187
```
**_Note_**: Username is your **fullname** in all **small case**. **Password** is same as **username**.

---

## **1. Verify Your Cluster**

```bash
kubectl cluster-info
kubectl get --raw='/readyz?verbose'
kubectl config -h
kubectl config get-contexts
kubectl auth can-i get nodes
```

<!-- notes: 5 mins. If someone gets connection error → check kubeconfig. -->

---

## **2. Running your First pod**

```bash
kubectl run web --image=nginx
kubectl get pod web
kubectl get pod web -o wide
kubectl describe pod web
```

---

## **3. Declarative vs Imperative**

```bash
kubectl get deploy web-deployment --dry-run=client -o yaml > exported.yaml
k apply -f exported.yaml

```
