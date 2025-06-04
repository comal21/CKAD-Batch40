## Multi-container
Switch to root user
```
sudo su
```
### Task 1: Sidecar container
```
vi sidecar.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-pod
spec:
  containers:
  - name: main-container
    image: nginx:latest
    ports:
    - containerPort: 80
    # Main application container

  - name: sidecar-container
    image: debian:latest
    command: ['sh', '-c', 'apt update && apt install curl -y && while true; do echo "Sidecar Running"; sleep 10; done']
    # Sidecar container
```
```	
kubectl apply -f sidecar.yaml
```
```
kubectl get pod
```
```
kubectl exec -it sidecar-pod -c sidecar-container -- sh
```
``` 
curl 'http://localhost:80/'
```
```
exit
```


### Task 2: Init container
```
vi init.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
spec:
  containers:
  - name: main-container
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: workdir
      mountPath: /app
    # Main application container

  initContainers:
  - name: init-container
    image: busybox:latest
    command: ['sh', '-c', 'echo "Init Container Completed" > /work-dir/completed.txt']
    volumeMounts:
    - name: workdir
      mountPath: /work-dir
    # Init container
  volumes:
  - name: workdir
    emptyDir: {}

```
```	
kubectl apply -f init.yaml
```
```
kubectl get pod
```


Go and update the command for the init container to write the file after a lapse of 60 seconds. You would notice the main container starting only after init container has completed its task
```
command: ['sh', '-c', 'sleep 60 && echo "Init Container Completed" > /work-dir/completed.txt']
```
Replace the pod
```
kubectl replace -f init.yaml --force
```
```
kubectl get po
```
Notice the PodInitializing status coming up only after a lapse of 60 seconds 

```
kubectl exec -it init-pod -c main-container -- bash
```
```
cd /app && ls
```
```
cat completed.txt
```
```
exit
```
#### Cleanup
```
kubectl delete po --all -n default
```
