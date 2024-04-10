# Kubernetes Pod



# what is Pod in Kubernetes

* Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.
* A Pod is a group of one or more containers.


## Creating the POD

### Create the Pod.yml file
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

