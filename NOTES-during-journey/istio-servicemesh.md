## [ISTIO-SERVICEMESH]:
--------
**WARNINGS** for performing such an excerise in future:

1> Refer official `istioctl` docs to install all istio components instead of helm to save some pain
2> KIND clusters in EC2 machine setup do not have their own load balancer, hence application SHOULD NOT BE though of accessing in local/other m/c web browser.
    For this excercise, I used `curl -s` command inside EC2 terminal to access the app


alias hl=helm       in      vim ~./bashrc

(add `istio helm charts` & update them - run both together)
--------------------
hl repo add <helm-release-name> <repo-url>
ex: helm repo add istio https://istio-release.storage.googleapis.com/charts

hl repo update

        "istio" has been added to your repositories
        Hang tight while we grab the latest from your chart repositories...
        ...Successfully got an update from the "istio" chart repository
        Update Complete. ⎈Happy Helming!⎈


(list the helm repos)                                               [OPTIONAL]
hl repo ls          or,  hl repo list
        
        NAME 	URL                                                
        istio	https://istio-release.storage.googleapis.com/charts



(install 1st component i.e, `istio-base`)
hl install <helm-release-name> <helm-chart-name> -n <ns-name> --version <version-num> --create-namespace --set profile=<profile-name>

ex: hl install istio-base istio/base -n istio-system --version 1.18.2 --create-namespace --set profile=istio-practice

        NAME: istio-base
        LAST DEPLOYED: Sun Jun 15 12:48:33 2025
        NAMESPACE: istio-system
        STATUS: deployed
        REVISION: 1
        TEST SUITE: None
        NOTES:
        Istio base successfully installed!

        To learn more about the release, try:
        $ helm status istio-base
        $ helm get all istio-base


(install 2nd component i.e, istiod` aka the control plane)
hl install <helm-release-name> <helm-chart-name> -n <ns-name> --version <version-num> --set profile=<profile-name>
ex: hl install istiod istio/istiod -n istio-system --version 1.18.2 --set profile=istio-practice

        NAME: istiod
        LAST DEPLOYED: Sun Jun 15 12:50:50 2025
        NAMESPACE: istio-system
        STATUS: deployed
        REVISION: 1
        TEST SUITE: None
        NOTES:
        "istiod" successfully installed!

        To learn more about the release, try:
        $ helm status istiod
        $ helm get all istiod

        Next steps:
        * Deploy a Gateway: https://istio.io/latest/docs/setup/additional-setup/gateway/
        * Try out our tasks to get started on common configurations:
            * https://istio.io/latest/docs/tasks/traffic-management
            * https://istio.io/latest/docs/tasks/security/
            * https://istio.io/latest/docs/tasks/policy-enforcement/
            * https://istio.io/latest/docs/tasks/policy-enforcement/
        * Review the list of actively supported releases, CVE publications and our hardening guide:
            * https://istio.io/latest/docs/releases/supported-releases/
            * https://istio.io/latest/news/security/
            * https://istio.io/latest/docs/ops/best-practices/security/

        For further documentation see https://istio.io website



(install 3rd component i.e, `istio-ingress` aka the istio gateway)
hl install <helm-release-name> <helm-chart-name> -n <ns-name> --version <version-num> --create-namespace
ex: hl install istio-ingress istio/gateway -n istio-ingress --version 1.18.2 --create-namespace

        NAME: istio-ingress
        LAST DEPLOYED: Sun Jun 15 13:03:28 2025
        NAMESPACE: istio-ingress
        STATUS: deployed
        REVISION: 1
        TEST SUITE: None
        NOTES:
        "istio-ingress" successfully installed!

        To learn more about the release, try:
        $ helm status istio-ingress
        $ helm get all istio-ingress

        Next steps:
        * Deploy an HTTP Gateway: https://istio.io/latest/docs/tasks/traffic-management/ingress/ingress-control/
        * Deploy an HTTPS Gateway: https://istio.io/latest/docs/tasks/traffic-management/ingress/secure-ingress/


(see what things are installed by `helm` in all namespaces)
hl ls -A

    NAME         	NAMESPACE    	REVISION	UPDATED                                	STATUS  	CHART         	APP VERSION
    istio-base   	istio-system 	1       	2025-06-15 12:48:33.27057728 +0000 UTC 	deployed	base-1.18.2   	1.18.2     
    istio-ingress	istio-ingress	1       	2025-06-15 13:03:28.43996686 +0000 UTC 	deployed	gateway-1.18.2	1.18.2     
    istiod       	istio-system 	1       	2025-06-15 12:50:50.456824757 +0000 UTC	deployed	istiod-1.18.2 	1.18.2    


(enable `istio side car` or `envoy proxy injection` on a specific/working namespace)
kl label namespace <ns-name> istio-injection=enabled
ex: kl label namespace stage-1-practicals istio-injection=enabled

        namespace/stage-1-practicals labeled

(NOTE: to disable the label, run)
kl label namespace <ns-name> istio-injection= --overwrite
ex: kl label namespace stage-1-practicals istio-injection= --overwrite


(verify the label, -L flag adds a column  `istio-injection` to see what it stored)
kl get ns -L istio-injection

    NAME                   STATUS   AGE     ISTIO-INJECTION
    default                Active   6d20h   
    ingress-nginx          Active   4d16h   
    istio-ingress          Active   91s     
    istio-system           Active   16m     
    kube-node-lease        Active   6d20h   
    kube-public            Active   6d20h   
    kube-system            Active   6d20h   
    kubernetes-dashboard   Active   6d20h   
    local-path-storage     Active   6d20h   
    stage-1-practicals     Active   5d3h    enabled


(also, we can see what `labels` are applied to a certain namespace)                      [OPTIONAL]
kl get namespace <ns-name> --show-labels
ex: kl get namespace stage-1-practicals --show-labels

    NAME                 STATUS   AGE    LABELS
    stage-1-practicals   Active   5d3h   istio-injection=enabled,kubernetes.io/metadata.name=stage-1-practicals


(check status of a istio component)                [OPTIONAL]
hl status <helm-release-name> -n <ns-name>         (this <helm-release-name> aka `istiod` is indeed a deployment)
ex: hl status istiod -n istio-system


Check istiod deployment is successfully installed and its pods are running,              [OPTIONAL]
kl get deployments -n <ns-name> --output wide
ex: kl get deployments -n istio-system -o wide

    NAME     READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                         SELECTOR
    istiod   1/1     1            1           23m   discovery    docker.io/istio/pilot:1.18.2   istio=pilot


or, more detailed info          [I PREFER THIS - see everything in the <ns-name> `istio-system`]
kl get all -n istio-system -o wide

    NAME                          READY   STATUS    RESTARTS   AGE   IP           NODE                                 NOMINATED NODE   READINESS GATES
    pod/istiod-67c8d87b7d-pdl2r   1/1     Running   0          27m   10.244.1.4   my-practicals-kind-cluster-worker2   <none>           <none>

    NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                                 AGE   SELECTOR
    service/istiod   ClusterIP   10.96.104.143   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP   27m   app=istiod,istio=pilot

    NAME                     READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                         SELECTOR
    deployment.apps/istiod   1/1     1            1           27m   discovery    docker.io/istio/pilot:1.18.2   istio=pilot

    NAME                                DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                         SELECTOR
    replicaset.apps/istiod-67c8d87b7d   1         1         1       27m   discovery    docker.io/istio/pilot:1.18.2   istio=pilot,pod-template-hash=67c8d87b7d

    NAME                                         REFERENCE           TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
    horizontalpodautoscaler.autoscaling/istiod   Deployment/istiod   cpu: 0%/80%   1         5         1          27m



## Getting the `istio deployment` manifests installed:

(clone the `isitio` git repo but only the folder named `samples` which is useful for us),

git clone --depth 1 --filter=blob:none --sparse https://github.com/istio/istio.git
cd istio
git sparse-checkout set samples/bookinfo
mv samples ../
cd ..
rm -rf istio

(one-liner --> I PREFER THIS)
git clone --depth 1 --filter=blob:none --sparse https://github.com/istio/istio.git && cd istio && git sparse-checkout set samples/bookinfo && mv samples ../ && cd .. && rm -rf istio


## Deploy your application using the kl command (contains multiple svc's):
kl apply -f <file.yaml>
ex: kl apply -f samples/bookinfo/platform/kube/bookinfo.yaml

        service/details created
        serviceaccount/bookinfo-details created
        deployment.apps/details-v1 created
        service/ratings created
        serviceaccount/bookinfo-ratings created
        deployment.apps/ratings-v1 created
        service/reviews created
        serviceaccount/bookinfo-reviews created
        deployment.apps/reviews-v1 created
        deployment.apps/reviews-v2 created
        deployment.apps/reviews-v3 created
        service/productpage created
        serviceaccount/bookinfo-productpage created
        deployment.apps/productpage-v1 created


## Create an Istio Gateway using the following command:				(shubham's bahari duniya)
kl apply -f <file.yaml> -n <ns-name>
ex: kl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml           or,     (more robust) kl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml -n stage-1-practicals

    if error appears as,

    resource mapping not found for name: "bookinfo-gateway" namespace: "" from "samples/bookinfo/networking/bookinfo-gateway.yaml": no matches for kind "Gateway" in version "networking.istio.io/v1"
    ensure CRDs are installed first
    resource mapping not found for name: "bookinfo" namespace: "" from "samples/bookinfo/networking/bookinfo-gateway.yaml": no matches for kind "VirtualService" in version "networking.istio.io/v1"
    ensure CRDs are installed first


    this is due to the bad luck for not installing with `istioctl` official docs but helm.......this error means, we need to update the few fields of file "samples/bookinfo/networking/bookinfo-gateway.yaml",

    1> 
    selector:
        istio: ingressgateway			to

    selector:
        istio: ingress


    # Update the selector to match your istio-ingress installation, run,
    sed -i 's/istio: ingressgateway/istio: ingress/g' samples/bookinfo/networking/bookinfo-gateway.yaml

    # Verify the change
    grep -A 2 "selector:" samples/bookinfo/networking/bookinfo-gateway.yaml

    selector:
        istio: ingress # use istio default controller
    servers:


    2> 
    apiVersion: networking.istio.io/v1		to
    apiVersion: networking.istio.io/v1alpha3

    # Update the apiVersion, run
    sed -i 's/networking.istio.io\/v1/networking.istio.io\/v1alpha3/g' samples/bookinfo/networking/bookinfo-gateway.yaml

    # Verify the change
    grep "apiVersion:" samples/bookinfo/networking/bookinfo-gateway.yaml
        apiVersion: networking.istio.io/v1alpha3
        apiVersion: networking.istio.io/v1alpha3


then, istio gateway should be created this time,
kl apply -f <file.yaml> -n <ns-name>
ex: kl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml -n stage-1-practicals

	gateway.networking.istio.io/bookinfo-gateway created
	virtualservice.networking.istio.io/bookinfo created



## Annotate the Istio Gateway:	
kl annotate gateway bookinfo-gateway networking.istio.io/service-type=ClusterIP -n stage-1-practicals

	gateway.networking.istio.io/bookinfo-gateway annotated


## see **gateway** and **virualservice** details:
(i) kl get gw -n <ns-name>	                    	    [gw=gateway]
ex: kl get gateway -n stage-1-practicals		

    NAME               AGE
    bookinfo-gateway   5s


or more details of gateway using `describe` command,

kl describe gw <gw-name>
ex: kl describe gateway bookinfo-gateway

        Name:         bookinfo-gateway
        Namespace:    stage-1-practicals
        Labels:       <none>
        Annotations:  networking.istio.io/service-type: ClusterIP
        API Version:  networking.istio.io/v1beta1
        Kind:         Gateway
        Metadata:
        Creation Timestamp:  2025-06-15T16:36:41Z
        Generation:          1
        Resource Version:    171961
        UID:                 4d77045f-c099-4760-a0ec-37e9d24e4a4c
        Spec:
        Selector:
            Istio:  ingress
        Servers:
            Hosts:
            *
            Port:
            Name:      http
            Number:    8080
            Protocol:  HTTP
        Events:          <none>


(ii) kl get vs -n <ns-name>	                        [vs=virtualservice]
ex: kl get virtualservice -n stage-1-practicals	

    NAME       GATEWAYS               HOSTS   AGE
    bookinfo   ["bookinfo-gateway"]   ["*"]   10s


====================================================[OPTIONAL]====can-be-avoided-if-followed-above================================
# Check if Istio CRDs are installed
kl get crd | grep istio

# Check specifically for Gateway and VirtualService CRDs
kl get crd gateways.networking.istio.io
kl get crd virtualservices.networking.istio.io

# Verify if virtualservice & gateway are created
kl get vs -n stage-1-practicals                         [vs=virtualservice]
kl get gw -n stage-1-practicals                    [gw=gateway]


# Debug your current istio-base installation:
hl get all istio-base -n istio-system
hl status istio-base -n istio-system


# Check the API version in your gateway file i.e, Check what API version the gateway file is using
head -10 samples/bookinfo/networking/bookinfo-gateway.yaml                  (uses `apiVersion: networking.istio.io/v1`)


# Check your actual istio-ingress gateway labels:
kl get pods -n istio-ingress --show-labels

# And check the service
kl get svc -n istio-ingress --show-labels

# Or, check what selector to use i.e, find the correct selector for your istio-ingress
kl get deployment -n istio-ingress -o yaml | grep -A 10 "matchLabels"


SOLUTION: Perfect! Now I can see your istio-ingress gateway labels. The selector should be istio: ingress (not istio: ingressgateway).


# Update the selector to match your istio-ingress installation
sed -i 's/istio: ingressgateway/istio: ingress/g' samples/bookinfo/networking/bookinfo-gateway.yaml

(basically, spec:
  # The selector matches the ingress gateway pod labels.
  # If you **installed Istio using Helm** following the standard documentation, this would be "istio=ingress"
  selector:
    istio: ingressgateway                   # use istio default controller

should change to,

  selector:
    istio: ingress                          # use helm default controller


# Verify the change
grep -A 2 "selector:" samples/bookinfo/networking/bookinfo-gateway.yaml

  selector:
    istio: ingress # use istio default controller
  servers:


then check the apiversions in both booking info yaml & CRD's:

kl get crd gateways.networking.istio.io -o yaml | grep -A 5 versions:

  versions:
  - name: v1alpha3
    schema:
      openAPIV3Schema:
        properties:
          spec:
  versions:
  - additionalPrinterColumns:
    - description: The names of gateways and sidecars that should apply these routes
      jsonPath: .spec.gateways
      name: Gateways
      type: string


grep "apiVersion:" samples/bookinfo/networking/bookinfo-gateway.yaml

        apiVersion: networking.istio.io/v1
        apiVersion: networking.istio.io/v1



========================================================================================================================

## Now, from **[istio official docs](https://istio.io/latest/docs/tasks/traffic-management/ingress/ingress-control/#determining-the-ingress-ip-and-ports):**

If you installed Istio using Helm, the ingress gateway name and namespace are both istio-ingress:

    $ export INGRESS_NAME=istio-ingress
    $ export INGRESS_NS=istio-ingress


Run the following command to determine if your Kubernetes cluster is in an environment that supports external load balancers:

kl get svc "$INGRESS_NAME" -n "$INGRESS_NS"

    NAME            TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                                      AGE
    istio-ingress   LoadBalancer   10.96.18.215   <pending>     15021:31082/TCP,80:32171/TCP,443:31775/TCP   6h21m

If the EXTERNAL-IP value is set, your environment has an external load balancer that you can use for the ingress gateway. If the EXTERNAL-IP value is <none> (or perpetually i.e, constantly <pending>), your environment does not provide an external load balancer for the ingress gateway.

If your environment does not support external load balancers, you can try **[accessing the ingress gateway using node ports](https://istio.io/latest/docs/tasks/traffic-management/ingress/ingress-control/#using-node-ports-of-the-ingress-gateway-service)**. Otherwise, set the ingress IP and ports using the following commands:

If your environment does not support external load balancers, you can still experiment with some of the Istio features by using the istio-ingressgateway service’s node ports.

Set the ingress ports:
```sh
export INGRESS_PORT=$(kl -n "${INGRESS_NS}" get service "${INGRESS_NAME}" -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
export SECURE_INGRESS_PORT=$(kl -n "${INGRESS_NS}" get service "${INGRESS_NAME}" -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
export TCP_INGRESS_PORT=$(kl -n "${INGRESS_NS}" get service "${INGRESS_NAME}" -o jsonpath='{.spec.ports[?(@.name=="tcp")].nodePort}')
```

Setting the ingress IP depends on the cluster provider:

4> Other environments:

export INGRESS_HOST=$(kl get po -l istio=ingress -n "${INGRESS_NS}" -o jsonpath='{.items[0].status.hostIP}')           (`istio=ingress` here since helm installed and changed the `selectors` in "samples/bookinfo/platform/kube/bookinfo.yaml", that's why)

Inspect the values of the INGRESS_HOST and INGRESS_PORT environment variables. Make sure they have valid values, according to the output of the following commands:

```sh
kl get svc -n istio-system
echo "INGRESS_HOST=$INGRESS_HOST, INGRESS_PORT=$INGRESS_PORT"
```
    NAME     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                                 AGE
    istiod   ClusterIP   10.96.104.143   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP   6h41m
    INGRESS_HOST=172.18.0.3, INGRESS_PORT=32171


Then return back to **[/examples/bookinfo/ docs url](https://istio.io/latest/docs/examples/bookinfo/)**

Set GATEWAY_URL:

    $ export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT


echo $GATEWAY_URL
        
        172.18.0.3:32171


Finally, hit/access the app inside the EC2/AWS env (nodeport svc),

curl -s "http://${GATEWAY_URL}/productpage" | grep -o "<title>.*</title>"

    <title>Simple Bookstore App</title>



or. make an API call,

curl -I http://$GATEWAY_URL/productpage

        HTTP/1.1 200 OK
        server: istio-envoy
        date: Sun, 15 Jun 2025 19:36:52 GMT
        content-type: text/html; charset=utf-8
        content-length: 15068
        vary: Cookie
        x-envoy-upstream-service-time: 197

For visual o/p, see snippets saved in "/Volumes/Seagate 1TB/Quick-save-folders/ML0ps-preparation/DevOps_routines_set/(PROJECT)_K8s_grind" folder:



