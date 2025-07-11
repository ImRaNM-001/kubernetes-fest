## [CLUSTERS]:
--------
(create cluster)
kind create cluster --name my-practicals-kind-cluster --config multi-node-kind-cluster-config.yaml

or, kind create cluster --name=my-practicals-kind-cluster --config=multi-node-kind-cluster-config.yaml


(see all clusters)               [BEST COMMAND]
kl config get-contexts

    CURRENT   NAME                                                                   CLUSTER                                     AUTHINFO                                                               NAMESPACE
*         hf-samsum-data-ingest-user@instana-tier-cluster.ap-south-1.eksctl.io   instana-tier-cluster.ap-south-1.eksctl.io   hf-samsum-data-ingest-user@instana-tier-cluster.ap-south-1.eksctl.io   
          kind-my-practicals-kind-cluster                                        kind-my-practicals-kind-cluster             kind-my-practicals-kind-cluster                                        helm-mania


(view current clusters - only KIND specific)
kind get clusters
	--> No kind clusters found.
    --> my-practicals-kind-cluster      (if clusters exist)


(view current clusters)
kl config current-context
	--> error: current-context is not set	(if clusters don't exist)
	--> kind-single-node-cluster	(if a `KIND` cluster exist)
    --> hf-samsum-data-ingest-user@instana-tier-cluster.ap-south-1.eksctl.io        (if a `EKS` cluster exist)
	

(switch to a specific cluster)
kl config use-context <cluster-name>
ex: kl config use-context kind-my-practicals-kind-cluster

ex: kl config use-context hf-samsum-data-ingest-user@instana-tier-cluster.ap-south-1.eksctl.io

    Switched to context "hf-samsum-data-ingest-user@instana-tier-cluster.ap-south-1.eksctl.io".


(see master & worker nodes)
docker ps | grep -i <cluster-name>
ex: docker ps | grep -i my-practicals-kind-cluster

    --> 5fde6afd6041   kindest/node:v1.32.5   "/usr/local/bin/entr…"   9 minutes ago   Up 8 minutes                                                                      my-practicals-kind-cluster-worker
        663d9875ecc9   kindest/node:v1.32.5   "/usr/local/bin/entr…"   9 minutes ago   Up 8 minutes   127.0.0.1:41447->6443/tcp                                          my-practicals-kind-cluster-control-plane
        be5e40208053   kindest/node:v1.32.5   "/usr/local/bin/entr…"   9 minutes ago   Up 8 minutes   0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp, 0.0.0.0:8080->8080/tcp   my-practicals-kind-cluster-worker2



(see cluster information for a specific Kubernetes context)
kl cluster-info

or, kl cluster-info --context kind-<cluster-name>
ex: kl cluster-info --context kind-my-practicals-kind-cluster


(delete cluster)
kind delete cluster --name my-practicals-kind-cluster


(see all nodes)             [BEST COMMAND]
kl get nodes

NAME                                       STATUS   ROLES           AGE   VERSION
my-practicals-kind-cluster-control-plane   Ready    control-plane   19m   v1.32.5
my-practicals-kind-cluster-worker          Ready    <none>          19m   v1.32.5
my-practicals-kind-cluster-worker2         Ready    <none>          19m   v1.32.5

