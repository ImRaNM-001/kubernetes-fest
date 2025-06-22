## [KUBECTL-PORT-FORWARD]:
--------

## 1. create a namespace 1st (for isolation)
kl create ns <ns-name>
ex: kl create ns argocd-ex
        
        namespace/argocd-ex created

## 2. switch to the namespace
kl config set-context --current --namespace <ns-name>
ex: kl config set-context --current --namespace argocd-ex

        Context "kind-my-practicals-kind-cluster" modified.


## 3. check current namespace
kl config view --minify | grep namespace
    
        namespace: argocd-ex


## 4. install `argocd` via official **yaml manifest**
kl apply -n <ns-name> -f <file-name.yaml>
ex: kl apply -n argocd-ex -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

        customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created
        customresourcedefinition.apiextensions.k8s.io/applicationsets.argoproj.io created
        customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io created
        serviceaccount/argocd-application-controller created
        serviceaccount/argocd-applicationset-controller created
        serviceaccount/argocd-dex-server created
        serviceaccount/argocd-notifications-controller created
        serviceaccount/argocd-redis created
        serviceaccount/argocd-repo-server created
        serviceaccount/argocd-server created
        role.rbac.authorization.k8s.io/argocd-application-controller created
        role.rbac.authorization.k8s.io/argocd-applicationset-controller created
        role.rbac.authorization.k8s.io/argocd-dex-server created
        role.rbac.authorization.k8s.io/argocd-notifications-controller created
        role.rbac.authorization.k8s.io/argocd-redis created
        role.rbac.authorization.k8s.io/argocd-server created
        clusterrole.rbac.authorization.k8s.io/argocd-application-controller created
        clusterrole.rbac.authorization.k8s.io/argocd-applicationset-controller created
        clusterrole.rbac.authorization.k8s.io/argocd-server created
        rolebinding.rbac.authorization.k8s.io/argocd-application-controller created
        rolebinding.rbac.authorization.k8s.io/argocd-applicationset-controller created
        rolebinding.rbac.authorization.k8s.io/argocd-dex-server created
        rolebinding.rbac.authorization.k8s.io/argocd-notifications-controller created
        rolebinding.rbac.authorization.k8s.io/argocd-redis created
        rolebinding.rbac.authorization.k8s.io/argocd-server created
        clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller created
        clusterrolebinding.rbac.authorization.k8s.io/argocd-applicationset-controller created
        clusterrolebinding.rbac.authorization.k8s.io/argocd-server created
        configmap/argocd-cm created
        configmap/argocd-cmd-params-cm created
        configmap/argocd-gpg-keys-cm created
        configmap/argocd-notifications-cm created
        configmap/argocd-rbac-cm created
        configmap/argocd-ssh-known-hosts-cm created
        configmap/argocd-tls-certs-cm created
        secret/argocd-notifications-secret created
        secret/argocd-secret created
        service/argocd-applicationset-controller created
        service/argocd-dex-server created
        service/argocd-metrics created
        service/argocd-notifications-controller-metrics created
        service/argocd-redis created
        service/argocd-repo-server created
        service/argocd-server created
        service/argocd-server-metrics created
        deployment.apps/argocd-applicationset-controller created
        deployment.apps/argocd-dex-server created
        deployment.apps/argocd-notifications-controller created
        deployment.apps/argocd-redis created
        deployment.apps/argocd-repo-server created
        deployment.apps/argocd-server created
        statefulset.apps/argocd-application-controller created
        networkpolicy.networking.k8s.io/argocd-application-controller-network-policy created
        networkpolicy.networking.k8s.io/argocd-applicationset-controller-network-policy created
        networkpolicy.networking.k8s.io/argocd-dex-server-network-policy created
        networkpolicy.networking.k8s.io/argocd-notifications-controller-network-policy created
        networkpolicy.networking.k8s.io/argocd-redis-network-policy created
        networkpolicy.networking.k8s.io/argocd-repo-server-network-policy created
        networkpolicy.networking.k8s.io/argocd-server-network-policy created


## 5. get the `argocd` services running,
kl get svc -n <ns-name>
ex: kl get svc -n argocd-ex

        NAME                                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
        argocd-applicationset-controller          ClusterIP   10.96.74.67     <none>        7000/TCP,8080/TCP            75s
        argocd-dex-server                         ClusterIP   10.96.73.62     <none>        5556/TCP,5557/TCP,5558/TCP   75s
        argocd-metrics                            ClusterIP   10.96.14.72     <none>        8082/TCP                     75s
        argocd-notifications-controller-metrics   ClusterIP   10.96.76.91     <none>        9001/TCP                     75s
        argocd-redis                              ClusterIP   10.96.19.79     <none>        6379/TCP                     75s
        argocd-repo-server                        ClusterIP   10.96.242.105   <none>        8081/TCP,8084/TCP            75s
        argocd-server                             ClusterIP   10.96.248.199   <none>        80/TCP,443/TCP               75s
        argocd-server-metrics                     ClusterIP   10.96.5.110     <none>        8083/TCP                     75s


## 6. retrieve the `argocd-server` UI login passwd,
kl get secret <secret-name> -n <ns-name> -o jsonpath="{.data.password}" | base64 -d
ex: kl get secret argocd-initial-admin-secret -n argocd-ex -o jsonpath="{.data.password}" | base64 -d

    -->  ig2XmfCueq0xBQyM


## 7. port-foward the `argocd-server` svc port to EC2 port,
kl port-forward <svc-name> <local-m/c-port>:<k8-svc-port> --address 0.0.0.0 -n <ns-name>

ex: kl port-forward svc/argocd-server 8443:80 --address 0.0.0.0 -n argocd-ex

or, kl port-forward svc/argocd-server :80 --address 0.0.0.0 -n argocd-ex               (if left blank, system auto assigns a port)
        
        Forwarding from 0.0.0.0:8443 -> 8080
        Handling connection for 8443
        Handling connection for 8443
        Handling connection for 8443
        Handling connection for 8443
        Handling connection for 8443
        Handling connection for 8443
        Handling connection for 8443
        Handling connection for 8443
        

## 8. Finally, cleanup all resources,
kl delete all --all -n <ns-name>
ex: kl delete all --all -n argocd-ex 

        pod "argocd-application-controller-0" deleted
        pod "argocd-applicationset-controller-655cc58ff8-wc2wf" deleted
        pod "argocd-dex-server-7d9dfb4fb8-czv2r" deleted
        pod "argocd-notifications-controller-6c6848bc4c-qkctv" deleted
        pod "argocd-redis-656c79549c-p6htk" deleted
        pod "argocd-repo-server-856b768fd9-jpb2m" deleted
        pod "argocd-server-99c485944-r7c9q" deleted
        service "argocd-applicationset-controller" deleted
        service "argocd-dex-server" deleted
        service "argocd-metrics" deleted
        service "argocd-notifications-controller-metrics" deleted
        service "argocd-redis" deleted
        service "argocd-repo-server" deleted
        service "argocd-server" deleted
        service "argocd-server-metrics" deleted
        deployment.apps "argocd-applicationset-controller" deleted
        deployment.apps "argocd-dex-server" deleted
        deployment.apps "argocd-notifications-controller" deleted
        deployment.apps "argocd-redis" deleted
        deployment.apps "argocd-repo-server" deleted
        deployment.apps "argocd-server" deleted
        replicaset.apps "argocd-server-99c485944" deleted
        statefulset.apps "argocd-application-controller" deleted




