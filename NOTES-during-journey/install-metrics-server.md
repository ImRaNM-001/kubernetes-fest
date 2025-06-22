## [INSTALL-`METRICS-SERVER`]:
--------
kl top node
        --> error: Metrics API not available

**Metrics API not available** means the **metrics-server** is not installed in the Kind cluster.

## **Install metrics-server:**
```bash
kl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
        --> serviceaccount/metrics-server unchanged
        clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader unchanged
        clusterrole.rbac.authorization.k8s.io/system:metrics-server unchanged
        rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader unchanged
        clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator unchanged
        clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server unchanged
        service/metrics-server unchanged
        deployment.apps/metrics-server created
        apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io unchanged


## **For Kind clusters, also patch it:**
```bash
kl patch deployment metrics-server -n kube-system --type='json' -p='[
  {"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"},
  {"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-preferred-address-types=InternalIP"}
]'
```
        --> deployment.apps/metrics-server patched

NOTE: The namespace is `kube-system`


## [VERY IMPORTANT-wait for the final success msg as below] **Wait for pod to restart:**                       
```bash
kl rollout status deployment metrics-server -n kube-system
```
        --> Waiting for deployment "metrics-server" rollout to finish: 1 old replicas are pending termination...
        Waiting for deployment "metrics-server" rollout to finish: 1 old replicas are pending termination...
        Waiting for deployment "metrics-server" rollout to finish: 1 old replicas are pending termination...
        deployment "metrics-server" successfully rolled out


## **Wait for 30 secs and test:**
```bash
# Wait for metrics-server to be ready   (OPTIONAL)
kl wait --for=condition=ready pod -l k8s-app=metrics-server -n kube-system --timeout=60s


# Test
kl top node

        NAME                                       CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)   
        my-practicals-kind-cluster-control-plane   95m          4%       956Mi           24%         
        my-practicals-kind-cluster-worker          163m         8%       370Mi           9%          
        my-practicals-kind-cluster-worker2         19m          0%       448Mi           11% 


kl top pod

        NAME                                                          CPU(cores)   MEMORY(bytes)   
        gh-actions-self-hosted-calc-app-deployment-6f7d597497-bqjnz   2m           53Mi            
        gh-actions-self-hosted-calc-app-deployment-6f7d597497-j26ch   2m           53Mi  

```

**The `--kubelet-insecure-tls` flag is needed for Kind clusters.**


