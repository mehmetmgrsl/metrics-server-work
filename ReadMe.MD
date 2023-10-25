# WIP

### Why metrics-server

- To collects resource metrics from Kubelets and exposes them in Kubernetes apiserver through Metrics API for use by Horizontal Pod Autoscaler and Vertical Pod Autoscaler (1*)

### Prerequisites

1. A K8S Cluster

- Generate a Custom CA Certificate:

   ```
   openssl genpkey -algorithm RSA -out custom-ca.key
   openssl req -new -key custom-ca.key -out custom-ca.csr
   openssl x509 -req -in custom-ca.csr -signkey custom-ca.key -out custom-ca.crt
   ```

- Start Minikube with the Custom CA Certificate:

  ```minikube start --driver='virtualbox' --nodes 2  --extra-config=apiserver.client-ca-file=custom-ca.crt --extra-config=apiserver.client-ca-key-file=custom-ca.key --kubernetes-version v1.25.14```

### Installation

1. helm repo add bitnami https://charts.bitnami.com/bitnami

2. helm upgrade --install metrics-server bitnami/metrics-server --create-namespace --namespace metrics-server --set apiService.create=true --set extraArgs='{--kubelet-insecure-tls=true,--kubelet-preferred-address-types=InternalIP}'

  *  --set apiService.create=true : metrics-server is going to register itself with API Services
  
     - kubectl get apiservices.apiregistration.k8s.io | grep -i metrics

        ```
        NAME                                     SERVICE                         AVAILABLE  
        v1beta1.metrics.k8s.io                   metrics-server/metrics-server   True        
        ```

### Test

```
kubectl top nodes

NAME                 CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
kind-control-plane   117m         1%     831Mi           5%        
kind-worker          41m          0%     432Mi           2%        
kind-worker2         28m          0%     347Mi           2%
```


- CPU(cores): The CPU usage in millicores (1/1000th of a CPU core).
- CPU%: The CPU usage as a percentage of the node's total CPU capacity.
- MEMORY(bytes): The memory usage in bytes.
- MEMORY%: The memory usage as a percentage of the node's total memory capacity.



### Resources

1* https://github.com/kubernetes-sigs/metrics-server

2* [Installing metrics-server | Jérôme Petazzoni LKE Workshop](https://www.youtube.com/watch?v=qUW1tsSXCkQ&t=299s)
