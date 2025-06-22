## [HELM-PACKAGING]:
--------
hl create <app-or-service-name>
hl create python-fastapi-app-helm       (creates following structure)

(display tree structure - folder view)
tree
.
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml


(list helm repos)
hl repo list

    NAME   	URL                                                
    istio  	https://istio-release.storage.googleapis.com/charts
    kyverno	https://kyverno.github.io/kyverno/                 


(search `charts` using helm)
hl search repo istio

    NAME               	CHART VERSION	APP VERSION	DESCRIPTION                                       
    istio/istiod       	1.26.2       	1.26.2     	Helm chart for istio control plane                
    istio/istiod-remote	1.23.6       	1.23.6     	Helm chart for a remote cluster using an extern...
    istio/ambient      	1.26.2       	1.26.2     	Helm umbrella chart for ambient                   
    istio/base         	1.26.2       	1.26.2     	Helm chart for deploying Istio cluster resource...
    istio/cni          	1.26.2       	1.26.2     	Helm chart for istio-cni components               
    istio/gateway      	1.26.2       	1.26.2     	Helm chart for deploying Istio gateways           
    istio/ztunnel      	1.26.2       	1.26.2     	Helm chart for istio ztunnel components 


or, (searching specifc - misses out column names)
hl search repo istio | grep istio

    istio/istiod       	1.26.2       	1.26.2     	Helm chart for istio control plane                
    istio/istiod-remote	1.23.6       	1.23.6     	Helm chart for a remote cluster using an extern...
    istio/ambient      	1.26.2       	1.26.2     	Helm umbrella chart for ambient                   
    istio/base         	1.26.2       	1.26.2     	Helm chart for deploying Istio cluster resource...
    istio/cni          	1.26.2       	1.26.2     	Helm chart for istio-cni components               
    istio/gateway      	1.26.2       	1.26.2     	Helm chart for deploying Istio gateways           
    istio/ztunnel      	1.26.2       	1.26.2     	Helm chart for istio ztunnel components     



(list all releases in a namespace)
hl ls  
    
    NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION


(package a `helm-chart`)
hl package python-fastapi-app-helm

    Successfully packaged chart and saved it to: /home/ubuntu/k8s-cluster-kind/stage-1-practicals/python-fastapi-app-helm-0.1.0.tgz

(create index file)
hl repo index .

ls

    04-python-fastapi-app         RBAC                              deployment-imagePullSecrets.yaml  kyverno-ex               python-fastapi-app-helm-0.1.0.tgz
    1                             containerization-examples-ch-1.0  index.yaml                        port-forward-ex          statefulset-ex
    COPY-python-fastapi-app-helm  deploy-strategies                 istio-servicemesh                 python-fastapi-app-helm  taints-tolerations


## push the package as an OCI artifact to OCI Registry (Modern Approach)
(authenticate in the ghcr repository)
helm registry login ghcr.io -u <github-username>
ex: helm registry login ghcr.io -u johndoe-001
    
    Password: 
    Login Succeeded


(push the artifact to the ghcr repository)
hl push <filename.tgz> oci://ghcr.io/<github-username>

ex: hl push python-fastapi-app-helm-0.1.0.tgz oci://ghcr.io/johndoe-001

    Pushed: ghcr.io/johndoe-001/python-fastapi-app-helm:0.1.0
    Digest: sha256:4cbec2------877867889XXXXXXa8XXXXXXX




