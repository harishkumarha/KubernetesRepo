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
## Create the the Pod
```html
kubectl apply -f https://raw.githubusercontent.com/harishkumarha/KubernetesRepo/main/pods/pod.yml
```

### Working with the Pods

* kubectl get pods
* kubectl describe pods/POD_NAME
* kubectl exec -it POD_NAME bash
* kubectl logs POD_NAME 

# Refer the official Kubernetes documentaion for working with Pods

[Kuberneets documentation for Pod]("https://kubernetes.io/docs/concepts/workloads/pods/")
