## Step 1: PV will be created by the administrator (Instructor step)

Example file

```yaml
# The instructor created multiple PVs like this one:
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-student-1 
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  hostPath:
    path: "/mnt/data/student1"
```

## Step 2: Create the Persistent Volume Claim (PVC)

Since the pv has been already created, you just need to claim it. We will request 100Mi of storage from the local-storage class.

pvc.yaml
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-lab-pvc
spec:
  volumeName: pv-username  
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  resources:
    requests:
      storage: 100Mi
```

## Step 3: Mount the PVC to a Pod

filename: pvc-pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-storage-pod
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: storage-volume
  volumes:
    - name: storage-volume
      persistentVolumeClaim:
        claimName: my-lab-pvc
```

## Step 4: Write data to the storage

```bash
kubectl exec -it nginx-storage-pod -- sh -c 'echo "Hello from 100MB Persistent Storage! The data survived! My name is shahgnp" > /usr/share/nginx/html/index.html' # Change your name
```

## Step 5: Verify the data is there

```bash
kubectl exec -it nginx-storage-pod -- curl localhost
```

## Step 6: Destroy the pod

```bash
kubectl delete pod nginx-storage-pod
kubectl get pods # Verify if the pod is deleted or not
```

## Step 7: Recreate the pod with the same PVC

```
kubectl apply -f pod.yaml
```

## Step 8: Verify the data is there

```bash
kubectl exec -it nginx-storage-pod -- curl localhost
```

## Step 9: Check the file exists in the file system

Check in which node the pod is scheduled

```bash
kubectl get pod -owide # Get the node name
```

SSH in the node

```
ssh support@node-ip 
```

Check in the /mnt/data folder for your data

```bash
ls /mnt/data/username
```