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

<img width="401" alt="Screenshot 2021-05-26 at 16 04 16" src="https://user-images.githubusercontent.com/82048393/119684364-5b0fa800-be3c-11eb-8954-27088288ab1a.png">


Define a resource quota. ie: resource request CPU = 1 CPU:
```
cat quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
spec:
  hard:
    requests.cpu: "1"
```

Apply the quota to the namespace by using the following command:
```
kubectl apply -f quota.yaml --namespace demo
```

You can check whether the resource quota is applied successfully to the namespace by executing the following command:
```
kubectl describe namespace demo
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
