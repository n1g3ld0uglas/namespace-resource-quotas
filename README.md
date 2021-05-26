# namespace-resource-quotas
Resource Quotas for Namespaces help define the resource requests and limits available to all objects within the namespace.

By default, cloud providers or dufferent K8 flavours have standard limits applied to the namespace. 
On GKE, the 'cpu request' is set to 0.1 cores. You can check this via kubectl describe command.
```
kubectl describe namespace default
```

Let's see what happens when resource quotas are applied to a newly-created namespace
```
kubectl create namespace demo
```

In the case of AKS, no resource quota is enforced in your default namespace:

<img width="390" alt="Screenshot 2021-05-26 at 16 23 02" src="https://user-images.githubusercontent.com/82048393/119686936-b2af1300-be3e-11eb-8a37-792ba7a4ca0f.png">


Define a resource quota. ie: resource request CPU = 1 CPU:
```
cat  << EOF > quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
spec:
  hard:
    requests.cpu: "1"
EOF    
```

Apply the quota to the namespace by using the following command:
```
kubectl apply -f quota.yaml --namespace demo
```

You can check whether the resource quota is applied successfully to the namespace by executing the following command:
```
kubectl describe namespace demo
```

<img width="466" alt="Screenshot 2021-05-26 at 16 17 12" src="https://user-images.githubusercontent.com/82048393/119686096-e3427d00-be3d-11eb-94d9-3558fc06818e.png">

Let's proceed to build the 
```
cat  << EOF > stress.yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo
spec:
  containers:
    - name: demo
      image: polinux/stress
      command: ["stress"]
      arg: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
EOF 
```

This pod inititates a stress process that tries to allocate memory of 150M at stratup.
If no limits are specified within the .yaml specification, the pod runs without any issues:

```
kubectl create -f stress.yaml
```

Now, try to create 2 pods that use 1 CPU 
```
kubectl apply -f nginx-cpu-1.yaml --namespace demo
```

The 2nd request will fail with the following error
```
Error from server (Forbidden): error when creating "nginx-cpu-1.yaml": pods "demo-1" is forbidden: exceeded quota: compute-resources, requested: requests.cpu=1, used: requests.cpu=1, limited: requests.cpu=1
```

Resource limits ensure quality of service for namespaced Kubernetes objects.
