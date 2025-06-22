
## [NAMESPACES]:
--------
(create namespace)
kl create ns <ns-name>                      [IMPERATIVE WAY]
ex: kl create ns stage-1-practicals

or,
kl apply -f namespaces.yaml              [DECLARATIVE WAY]


(switch to namespace)
kl config set-context --current --namespace stage-1-practicals
or,
kl config set-context --current --namespace=default     (switches to `default` ns)

# Method 2: Use -n flag with each command everytime (if not switching)
kl get pods -n stage-1-practicals
kl apply -f deployment.yaml -n stage-1-practicals


(see all namespaces)
kl get ns
    NAME                   STATUS   AGE
    default                Active   2d1h
    kube-node-lease        Active   2d1h
    kube-public            Active   2d1h
    kube-system            Active   2d1h
    kubernetes-dashboard   Active   2d1h
    local-path-storage     Active   2d1h
    stage-1-practicals     Active   8h

kl get ns <ns-name>     (for specific ns)



(see current namespace)
# Method 1: Check current context namespace       [good o/p]
kl config view --minify | grep namespace        (will show blank for `default` namespace)

# Method 2: Get current context details
kl config get-contexts

# Method 3: Show current namespace directly         [bekar o/p]
kl config view --minify -o jsonpath='{..namespace}'

(see description of a ns in detail)
kl describe ns                       (for all ns)

or,
kl describe ns <ns-name>             (for specific ns)


(see everything i.e, pods, svc, ingress inside a namespace)
kl get all -n <ns-name>
        NAME                                      READY   STATUS    RESTARTS   AGE
        pod/fastapi-deployment-56bdbd747b-cmpdp   1/1     Running   0          27m
        pod/fastapi-deployment-56bdbd747b-rzcdj   1/1     Running   0          27m

        NAME                      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
        service/fastapi-service   ClusterIP   10.96.53.111   <none>        80/TCP    36m

        NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/fastapi-deployment   2/2     2            2           36m

        NAME                                            DESIRED   CURRENT   READY   AGE
        replicaset.apps/fastapi-deployment-56bdbd747b   2         2         2       36m


(delete namespace)
kl delete ns <ns-name>


(see resources like pods inside a ns)
kl get pods -o wide -n <ns-name>

or, (very detailed)

kl describe pods -n <ns-name>


