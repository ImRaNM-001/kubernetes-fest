## [DEPLOYMENT-STRATEGIES]:
--------------
(A) ROLLING UPDATE Strategy:

(deploy the yaml manifest)
kl apply -f [deploy-rolling-update.yaml](../stage-3-practicals/rolling-update-nginx/deploy-rolling-update.yaml) --record

    Flag --record has been deprecated, --record will be removed in the future
    deployment.apps/nginx-deployment created


(check the rollout status)
kl rollout status deployment.apps/nginx-deployment

    deployment "nginx-deployment" successfully rolled out


(see the deployment)
kl get deploy

    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   4/4     4            4           117s


(Apply Rolling Update - one of the ways, others are: `kl edit` & `kl apply` & `kl replace` commands)
kl set image deployment <deployment-name> <image-name>=<image-name>:<image-version>

ex: kl set image deployment nginx-deployment nginx=nginx:1.14.2

    deployment.apps/nginx-deployment image updated


(check history or deployment revisions - there will be `2 revisions` - one for `nginx:1.14.0` and other for `nginx:1.14.2`)
kl rollout history deployment <deployment-name>

ex: kl rollout history deployment nginx-deployment

    deployment.apps/nginx-deployment 
    REVISION  CHANGE-CAUSE
    1         kubectl apply --filename=deploy-rolling.yaml --record=true
    2         kubectl apply --filename=deploy-rolling.yaml --record=true


(rollback deployment to previous version - i.e, to `image: nginx:1.14.0`)
kl rollout undo deployment <deployment-name> --to-revision=1 

ex: kl rollout undo deployment nginx-deployment --to-revision=1         (pods get recreated again)

    deployment.apps/nginx-deployment rolled back


NOTE: if switched back to revision=2 again, then subsequent revisions would get created replaced/succeeded by #revision3 & #revision4 as,

    REVISION  CHANGE-CAUSE
    3         kubectl apply --filename=deploy-rolling.yaml --record=true
    4         kubectl apply --filename=deploy-rolling.yaml --record=true



## Q: will the rolling update even happen?

minReadySeconds: 5
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 0
    maxUnavailable: 0


Ans: No, the rolling update won’t proceed with maxSurge: 0 and maxUnavailable: 0.
These settings mean:

🚫 No extra pods can be added (maxSurge: 0)
🚫 No pods can go down during the update (maxUnavailable: 0)

👉 So the Deployment gets stuck, because it can’t make room to update pods.



====================================================================================
(B) CANARY DEPLOYMENT Strategy:
--------------

(deploy all yaml manifests)
kl apply -f .

    deployment.apps/canary created
    service/canary created
    deployment.apps/production created
    service/production created
    ingress.networking.k8s.io/canary created
    ingress.networking.k8s.io/production created


(check the ingress running in both `canary-v2` and `prod-v1` env)
kl get ing

    NAME         CLASS   HOSTS                    ADDRESS     PORTS   AGE
    canary       nginx   echo.prod.mydomain.com   localhost   80      110s
    production   nginx   echo.prod.mydomain.com   localhost   80      110s
    

(to get view of all resources running post deployment)
kl get all

    NAME                              READY   STATUS    RESTARTS   AGE
    pod/canary-7d6f76c456-k7t6z       1/1     Running   0          4m17s
    pod/production-648cdf566f-kkxtz   1/1     Running   0          4m17s

    NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
    service/canary       ClusterIP   10.96.63.52    <none>        80/TCP    34m
    service/production   ClusterIP   10.96.65.202   <none>        80/TCP    34m

    NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/canary       1/1     1            1           34m
    deployment.apps/production   1/1     1            1           34m

    NAME                                    DESIRED   CURRENT   READY   AGE
    replicaset.apps/canary-57cbb68d66       0         0         0       34m
    replicaset.apps/canary-7d6f76c456       1         1         1       4m17s
    replicaset.apps/production-648cdf566f   1         1         1       4m17s
    replicaset.apps/production-858484bfd4   0         0         0       34m



## In order to hit the application via **curl** call:
_______________________________________________________________________________________________________________
[WAY1 - easy]
docker ps

    CONTAINER ID   IMAGE                  COMMAND                  CREATED       STATUS          PORTS                                                              NAMES
    5fde6afd6041   kindest/node:v1.32.5   "/usr/local/bin/entr…"   12 days ago   Up 17 minutes                                                                      my-practicals-kind-cluster-worker

    663d9875ecc9   kindest/node:v1.32.5   "/usr/local/bin/entr…"   12 days ago   Up 17 minutes   127.0.0.1:41447->6443/tcp                                          my-practicals-kind-cluster-control-plane

    be5e40208053   kindest/node:v1.32.5   "/usr/local/bin/entr…"   12 days ago   Up 17 minutes   0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp, 0.0.0.0:8080->8080/tcp   my-practicals-kind-cluster-worker2


## Since worker2 (worker node 2) has 0.0.0.0:80->80/tcp mapping
for i in $(seq 1 20); do curl -s -H "Host: echo.prod.mydomain.com" localhost:80 | grep "Hostname"; done | sort

    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v2-canary
    Hostname: hello-from-app-v2-canary
    Hostname: hello-from-app-v2-canary
    Hostname: hello-from-app-v2-canary
    Hostname: hello-from-app-v2-canary
    Hostname: hello-from-app-v2-canary

NOTE: if ssh'd into the cluster for other K8 distributions, run below command:
for i in $(seq 1 20); do curl -s --resolve echo.prod.mydomain.com:80:$INGRESS_CONTROLLER_IP echo.prod.mydomain.com  | grep "Hostname"; done


## Or, Force new connections - Use different user-agents or headers to force new connections:
for i in $(seq 1 20); do curl -s -H "Host: echo.prod.mydomain.com" -H "User-Agent: test-$i" localhost:80 | grep "Hostname"; done | sort | uniq -c

      8 Hostname: hello-from-app-v2-canary
     12 Hostname: hello-from-app-v1-production


_______________________________________________________________________________________________________________
[WAY2 - bit complex]
pick control plane docker <container-id> from above, then do

docker exec -it 663d9875ecc9 /bin/bash

    root@my-practicals-kind-cluster-control-plane:/# kubectl get svc -n ingress-nginx
    NAME                                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
    ingress-nginx-controller             NodePort    10.96.5.216    <none>        80:32735/TCP,443:30627/TCP   10d
    ingress-nginx-controller-admission   ClusterIP   10.96.15.164   <none>        443/TCP                      10d


for i in $(seq 1 20); do curl -s --resolve <INGRESS-HOST>:<INGRESS-PORT>:<INGRESS-CONTROLLER-NODEPORT-IP> echo.prod.mydomain.com | grep "Hostname"; done | sort
ex: for i in $(seq 1 20); do curl -s --resolve echo.prod.mydomain.com:80:10.96.5.216 echo.prod.mydomain.com | grep "Hostname"; done | sort

    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v1-production
    Hostname: hello-from-app-v2-canary
    Hostname: hello-from-app-v2-canary
    Hostname: hello-from-app-v2-canary
    Hostname: hello-from-app-v2-canary
    Hostname: hello-from-app-v2-canary
    Hostname: hello-from-app-v2-canary


now increase canary traffic via,
kl edit ing canary                      (to change the  nginx.ingress.kubernetes.io/canary-weight: "10")  to "30"

