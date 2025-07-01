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
hl repo list        or, hl repo ls

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
    
    NAME            	NAMESPACE 	REVISION	UPDATED                                	STATUS  	CHART                        	APP VERSION
    helm-fastapi-app	helm-mania	1       	2025-06-27 10:35:33.507613249 +0000 UTC	deployed	python-fastapi-app-helm-0.1.0	1.16.0 


(package a `helm-chart`)
hl package python-fastapi-app-helm

    Successfully packaged chart and saved it to: /home/ubuntu/k8s-cluster-kind/stage-1-practicals/python-fastapi-app-helm-0.1.0.tgz

(create index file)
hl repo index .

> This step is **not required** for OCI registries like GHCR, but is needed if we want to serve our chart via a static HTTP Helm repo.

ls

    04-python-fastapi-app         RBAC                              deployment-imagePullSecrets.yaml  kyverno-ex               python-fastapi-app-helm-0.1.0.tgz
    1                             containerization-examples-ch-1.0  index.yaml                        port-forward-ex          statefulset-ex
    COPY-python-fastapi-app-helm  deploy-strategies                 istio-servicemesh                 python-fastapi-app-helm  taints-tolerations


## push the package as an OCI artifact to OCI Registry (Modern Approach)

(authenticate in the ghcr repository)
hl registry login ghcr.io -u <github-username>

ex: hl registry login ghcr.io -u johndoe-001
    
    Password: 
    Login Succeeded


(push the artifact to the ghcr repository)
hl push <filename.tgz> oci://ghcr.io/<github-username>

ex: hl push python-fastapi-app-helm-0.1.0.tgz oci://ghcr.io/johndoe-001

    Pushed: ghcr.io/johndoe-001/python-fastapi-app-helm:0.1.0
    Digest: sha256:4cbec2------877867889XXXXXXa8XXXXXXX

> uploads the chart as an OCI artifact, stored like a container image and thus looks like
    `docker pull ghcr.io/imranm-001/python-fastapi-app-helm:0.1.0`


## (HOW - others should run the Kubernetes app using the helm chart (k8 manifests) .tgz file):
hl registry login ghcr.io -u <github-username>

hl pull oci://ghcr.io/<github-username>/<chart-name> --version 0.1.0
ex: hl pull oci://ghcr.io/johndoe-001/python-fastapi-app-helm --version 0.1.0	    (pulls the tarball i.e, .tgz and then we extract to see manifests)

(to install chart)
hl install <release-name> ./<chart-name>            (<release-name> can be like `dev-release`, `staging-release`)
ex: hl install python-fastapi-app ./python-fastapi-app-helm

(or combining both commands)
hl install python-fastapi-app oci://ghcr.io/johndoe-001/python-fastapi-app-helm --version 0.1.0



(pull a helm chart from OCI registry ex: `ghcr.io`)
hl pull oci://ghcr.io/<github-username>/<chart-name> --version 0.1.0
ex: hl pull oci://ghcr.io/imranm-001/python-fastapi-app-helm --version 0.1.0

    Pulled: ghcr.io/imranm-001/python-fastapi-app-helm:0.1.0
    Digest: sha256:0614ba9cdedefb89d2be2ab113db12766342d0fe32ec65f2de551fbc1c5a50b5


(after OCI pull, see the artifact info via)
hl show all oci://ghcr.io/imranm-001/<chart-name> --version 0.1.0

ex: hl show all oci://ghcr.io/imranm-001/python-fastapi-app-helm --version 0.1.0

    Pulled: ghcr.io/imranm-001/python-fastapi-app-helm:0.1.0
    Digest: sha256:0614ba9cdedefb89d2be2ab113db12766342d0fe32ec65f2de551fbc1c5a50b5
    apiVersion: v2
    appVersion: 1.16.0
    description: A Helm chart for Python FastAPI application deployment on Kubernetes
    home: https://github.com/ImRaNM-001/helm-charts-py-fastapi-app
    keywords:
    - fastapi
    - python
    - api
    - microservice
    maintainers:
    - email: maintainer@example.com
    name: FastAPI Team
    name: python-fastapi-app-helm
    sources:
    - https://github.com/ImRaNM-001/helm-charts-py-fastapi-app
    type: application
    version: 0.1.0

    ---
    # Default values for python-fastapi-app-helm.
    # This is a YAML-formatted file.
    # Declare variables to be substituted into your templates.


or, (see only the chart)
hl show chart oci://ghcr.io/imranm-001/<chart-name> --version 0.1.0

ex: hl show chart oci://ghcr.io/imranm-001/python-fastapi-app-helm --version 0.1.0

    Pulled: ghcr.io/imranm-001/python-fastapi-app-helm:0.1.0
    Digest: sha256:0614ba9cdedefb89d2be2ab113db12766342d0fe32ec65f2de551fbc1c5a50b5
    apiVersion: v2
    appVersion: 1.16.0
    description: A Helm chart for Python FastAPI application deployment on Kubernetes
    home: https://github.com/ImRaNM-001/helm-charts-py-fastapi-app
    keywords:
    - fastapi
    - python
    - api
    - microservice
    maintainers:
    - email: maintainer@example.com
    name: FastAPI Team
    name: python-fastapi-app-helm
    sources:
    - https://github.com/ImRaNM-001/helm-charts-py-fastapi-app
    type: application
    version: 0.1.0


## To make the github repo a valid Helm chart repo:
- Go to the repo’s Settings → Pages.

- Under "Build and deployment", select the branch (e.g., main) and the folder (usually /root).

- Save and wait for (at least 5 mins) the GitHub Pages URL (e.g., https://imranm-001.github.io/helm-charts-py-fastapi-app/) to deploy.

- Sometimes it takes a few minutes for GitHub Pages to update and serve the files.

- Check GitHub Pages URL:
        Visit https://imranm-001.github.io/helm-charts-py-fastapi-app/index.yaml in the browser.
        If you get a 404, GitHub Pages is not serving the files yet.


- Then, use this URL with helm repo add:
hl repo add <repo-name> <repo-URL>
ex: hl repo add fastapi-charts https://imranm-001.github.io/helm-charts-py-fastapi-app/

    "fastapi-charts" has been added to your repositories


hl repo update

    Hang tight while we grab the latest from your chart repositories...
    ...Successfully got an update from the "fastapi-charts" chart repository
    ...Successfully got an update from the "kyverno" chart repository
    ...Successfully got an update from the "istio" chart repository
    Update Complete. ⎈Happy Helming!⎈


hl repo ls              (display all helm repositories)

    NAME          	URL                                                     
    istio         	https://istio-release.storage.googleapis.com/charts     
    kyverno       	https://kyverno.github.io/kyverno/                      
    fastapi-charts	https://imranm-001.github.io/helm-charts-py-fastapi-app/


hl search repo <repo-name>						(subset of helm repo ls if `repo name` is known)
ex: hl search repo fastapi-charts	

	NAME                                  	CHART VERSION	APP VERSION	DESCRIPTION                                       
	fastapi-charts/python-fastapi-app-helm	0.1.0        	1.16.0     	A Helm chart for Python FastAPI application dep...


(installing the chart)
hl install <release-name> <repo-name>/<chart-name>
ex: hl install helm-fastapi-app fastapi-charts/python-fastapi-app-helm

    NAME: helm-fastapi-app
    LAST DEPLOYED: Fri Jun 27 10:35:33 2025
    NAMESPACE: helm-mania
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    1. Get the application URL by running these commands:
    http://fastapi.local.org/



(show helm **values** command),
hl show values <repo-name>/<chart-name>

ex: hl show values fastapi-charts/python-fastapi-app-helm                       or, hl show values bitnami/nginx

        # Default values for python-fastapi-app-helm.
        # This is a YAML-formatted file.
        # Declare variables to be substituted into your templates.

        replicaCount: 2

        image:
        repository: 04-python-fastapi-app
        pullPolicy: Never
        tag: "latest"

        nameOverride: ""
        fullnameOverride: ""

        # Security Context
        securityContext:
        runAsNonRoot: true
        runAsUser: 1001

        containerSecurityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        runAsNonRoot: true
        runAsUser: 1001
        capabilities:
            drop:
            - ALL

        service:
        type: ClusterIP
        port: 80
        targetPort: 80
        protocol: TCP

        ingress:
        enabled: true
        className: "nginx"
        annotations:
            nginx.ingress.kubernetes.io/rewrite-target: /
        hosts:
            - host: fastapi.local.org
            paths:
                - path: /
                pathType: Prefix
        tls: []

        resources:
        limits:
            cpu: 500m
            memory: 128Mi
        requests:
            cpu: 250m
            memory: 64Mi

        # ConfigMap data
        config:
        dbPort: "8886"

        # Secret data (base64 encoded values)
        secrets:
        dbUsername: "YWRtaW4="              # admin
        dbPassword: "U3VwZXJTZWNyZXQxMjM="   # SuperSecret123
        dbHost: "bG9jYWxob3N0"              # localhost
        apiKey: "YWJjZGVmZ2hpams="           # abcdefghijk
        jwtSecret: "bXlfc3VwZXJfc2VjcmV0X2p3dF9rZXk="  # my_super_secret_jwt_key

        # Volume mounts
        volumeMounts:
        - name: db-env-port
            mountPath: /etc/config
            readOnly: true
        - name: secret-files
            mountPath: /mnt/secrets
            readOnly: true
        - name: tmp-volume
            mountPath: /tmp

        # Pod annotations
        podAnnotations: {}

        # Pod labels
        podLabels:
        app: fastapi-app

        # Node selector
        nodeSelector: {}

        # Tolerations
        tolerations: []

        # Affinity
        affinity: {}

        # Autoscaling
        autoscaling:
        enabled: false
        minReplicas: 1
        maxReplicas: 100
        targetCPUUtilizationPercentage: 80
        # targetMemoryUtilizationPercentage: 80

        # Additional volumes
        volumes: []

        # Additional volume mounts
        additionalVolumeMounts: []

        # Additional environment variables
        env: []

        # Service account
        serviceAccount:
        # Specifies whether a service account should be created
        create: true
        # Automatically mount a ServiceAccount's API credentials?
        automount: true
        # Annotations to add to the service account
        annotations: {}
        # The name of the service account to use.
        # If not set and create is true, a name is generated using the fullname template
        name: ""


(To preview the manifests (i.e, yaml files in /template folder) without deploying)	
hl template <release-name> <repo-name>/<chart-name>             (syntax similiarity with **helm install** command)

ex: hl template helm-fastapi-app fastapi-charts/python-fastapi-app-helm                     or, hl template my-nginx bitnami/nginx 

        ---
        # Source: python-fastapi-app-helm/templates/serviceaccount.yaml
        apiVersion: v1
        kind: ServiceAccount
        metadata:
        name: helm-fastapi-app-python-fastapi-app-helm
        labels:
            helm.sh/chart: python-fastapi-app-helm-0.1.0
            app.kubernetes.io/name: python-fastapi-app-helm
            app.kubernetes.io/instance: helm-fastapi-app
            app.kubernetes.io/version: "1.16.0"
            app.kubernetes.io/managed-by: Helm
        automountServiceAccountToken: true
        ---
        # Source: python-fastapi-app-helm/templates/service.yaml
        apiVersion: v1
        kind: Service
        metadata:
        name: helm-fastapi-app-python-fastapi-app-helm
        labels:
            helm.sh/chart: python-fastapi-app-helm-0.1.0
            app.kubernetes.io/name: python-fastapi-app-helm
            app.kubernetes.io/instance: helm-fastapi-app
            app.kubernetes.io/version: "1.16.0"
            app.kubernetes.io/managed-by: Helm
        spec:
        type: ClusterIP
        ports:
            - port: 80
            targetPort: 80
            protocol: TCP
            name: http
        selector:
            app.kubernetes.io/name: python-fastapi-app-helm
            app.kubernetes.io/instance: helm-fastapi-app
        ---
        # Source: python-fastapi-app-helm/templates/deployment.yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
        name: helm-fastapi-app-python-fastapi-app-helm
        labels:
            helm.sh/chart: python-fastapi-app-helm-0.1.0
            app.kubernetes.io/name: python-fastapi-app-helm
            app.kubernetes.io/instance: helm-fastapi-app
            app.kubernetes.io/version: "1.16.0"
            app.kubernetes.io/managed-by: Helm
        spec:
        replicas: 2
        selector:
            matchLabels:
            app.kubernetes.io/name: python-fastapi-app-helm
            app.kubernetes.io/instance: helm-fastapi-app
        template:
            metadata:
            annotations:
            labels:
                helm.sh/chart: python-fastapi-app-helm-0.1.0
                app.kubernetes.io/name: python-fastapi-app-helm
                app.kubernetes.io/instance: helm-fastapi-app
                app.kubernetes.io/version: "1.16.0"
                app.kubernetes.io/managed-by: Helm
                app: fastapi-app
            spec:
            serviceAccountName: helm-fastapi-app-python-fastapi-app-helm
            securityContext:
                runAsNonRoot: true
                runAsUser: 1001
            containers:
                - name: python-fastapi-app-helm
                securityContext:
                    allowPrivilegeEscalation: false
                    capabilities:
                    drop:
                    - ALL
                    readOnlyRootFilesystem: true
                    runAsNonRoot: true
                    runAsUser: 1001
                image: "04-python-fastapi-app:latest"
                imagePullPolicy: Never
                ports:
                    - name: http
                    containerPort: 80
                    protocol: TCP
                volumeMounts:
                    - name: db-env-port
                    mountPath: /etc/config
                    readOnly: true
                    - name: secret-files
                    mountPath: /mnt/secrets
                    readOnly: true
                    - name: tmp-volume
                    mountPath: /tmp
                    readOnly: 
                resources:
                    limits:
                    cpu: 500m
                    memory: 128Mi
                    requests:
                    cpu: 250m
                    memory: 64Mi
            volumes:
                - name: db-env-port
                configMap:
                    name: helm-fastapi-app-python-fastapi-app-helm-config
                - name: secret-files
                secret:
                    secretName: helm-fastapi-app-python-fastapi-app-helm-secret
                - name: tmp-volume
                emptyDir: {}
        ---
        # Source: python-fastapi-app-helm/templates/ingress.yaml
        apiVersion: networking.k8s.io/v1
        kind: Ingress
        metadata:
        name: helm-fastapi-app-python-fastapi-app-helm
        labels:
            helm.sh/chart: python-fastapi-app-helm-0.1.0
            app.kubernetes.io/name: python-fastapi-app-helm
            app.kubernetes.io/instance: helm-fastapi-app
            app.kubernetes.io/version: "1.16.0"
            app.kubernetes.io/managed-by: Helm
        annotations:
            nginx.ingress.kubernetes.io/rewrite-target: /
        spec:
        ingressClassName: nginx
        rules:
            - host: "fastapi.local.org"
            http:
                paths:
                - path: /
                    pathType: Prefix
                    backend:
                    service:
                        name: helm-fastapi-app-python-fastapi-app-helm
                        port:
                        number: 80


(search/look for **`helm charts`** in the Artifact Hub (a public Helm chart repository), which lists helm charts from dozens of different repositories)
hl search hub <chart-name>     

ex: hl search hub wordpress

    URL                                               	CHART VERSION	APP VERSION        	DESCRIPTION                                       
    https://artifacthub.io/packages/helm/kube-wordp...	0.1.0        	1.1                	this is my wordpress package                      
    https://artifacthub.io/packages/helm/wordpress-...	1.0.2        	1.0.0              	A Helm chart for deploying Wordpress+Mariadb st...
    https://artifacthub.io/packages/helm/bizlinked/...	25.0.0       	6.8.1              	WordPress is the world's most popular blogging ...
    https://artifacthub.io/packages/helm/bitnami/wo...	25.0.0       	6.8.1              	WordPress is the world's most popular blogging ...
    https://artifacthub.io/packages/helm/bitnami-ak...	15.2.13      	6.1.0              	WordPress is the world's most popular blogging ...



(see `helm repos` and `helm list` clubbed together)

echo "=== Helm Repositories ==="
helm repo list
echo ""
echo "=== Helm Releases ==="
helm list --output table

    === Helm Repositories ===
	NAME          	URL                                                     
	istio         	https://istio-release.storage.googleapis.com/charts     
	kyverno       	https://kyverno.github.io/kyverno/                      
	fastapi-charts	https://imranm-001.github.io/helm-charts-py-fastapi-app/

	=== Helm Releases ===
	NAME            	NAMESPACE 	REVISION	UPDATED                                	STATUS  	CHART                        	APP VERSION
	helm-fastapi-app	helm-mania	1       	2025-06-27 10:35:33.507613249 +0000 UTC	deployed	python-fastapi-app-helm-0.1.0	1.16.0  



(see `history` of releases)
hl history <release-name>				or, helm history RELEASE_NAME							

ex: hl history helm-fastapi-app

    REVISION	UPDATED                 	STATUS  	CHART                        	APP VERSION	DESCRIPTION     
    1       	Fri Jun 27 10:35:33 2025	deployed	python-fastapi-app-helm-0.1.0	1.16.0     	Install complete



(**--set** flag used to update **values** inside `values.yaml`)
hl upgrade <release-name> --set <some_key>=<some_value> <repo-name>/<chart-name>

ex: hl upgrade helm-fastapi-app --set image.pullPolicy=IfNotPresent fastapi-charts/python-fastapi-app-helm

	Release "helm-fastapi-app" has been upgraded. Happy Helming!
	NAME: helm-fastapi-app
	LAST DEPLOYED: Fri Jun 27 10:56:52 2025
	NAMESPACE: helm-mania
	STATUS: deployed
	REVISION: 2
	TEST SUITE: None
	NOTES:
	1. Get the application URL by running these commands:
	http://fastapi.local.org/


(see the changed aka `deployed values` in REVISION 2)
hl get values <release-name> -n <ns-name>                   [NOTE: `hl get` would display current release/deployed values whilst `hl show` the chart defaults]

ex: hl get values helm-fastapi-app -n helm-mania

	USER-SUPPLIED VALUES:
	image:
	pullPolicy: IfNotPresent


(rolling back the changes)
hl rollback <RELEASE_NAME> <REVISION_ID>
ex: hl rollback helm-fastapi-app 1
	
    Rollback was a success! Happy Helming!


(now do)
hl ls			            (now helm would create a new release `REVISION` --> 3)

	NAME            	NAMESPACE 	REVISION	UPDATED                                	STATUS  	CHART                        	APP VERSION
	helm-fastapi-app	helm-mania	3       	2025-06-27 11:05:02.826056758 +0000 UTC	deployed	python-fastapi-app-helm-0.1.0	1.16.0   



(Finally, CLEAN UP chart(s) in namespace),
--------------
hl uninstall <release-name> -n <ns-name>            or, simply  hl uninstall <release-name>
ex: hl uninstall helm-fastapi-app

    release "helm-fastapi-app" uninstalled

