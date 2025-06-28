## [DEPLOYMENTS]:
--------
kl apply -f deployment.yaml

kl get deploy -o wide

kl get rs -o wide       (shows wide o/p for `replica set`)

kl get pods -w      (watch for pods as they come up if accidentally deleted)

kl delete pod <pod-name>

kl edit deploy <deployment-name> -o json        (:wq! and the deployment auto applies)

kl delete deploy <deployment-name>      (deletes all pods)



## [EXCERCISE: 04-python-fastapi-app INGRESS practice commands]:
--------
kl get all -n $(kl get ns | grep stage | awk -F" " '{print $1}')
    --> No resources found in stage-1-practicals namespace.

# Method 1: Apply multiple files in one command
kl apply -f deployment.yaml -f service.yaml

# Method 2: Apply all YAML files in current directory
kl apply -f .

(CHECKING LOGS),
(for the `app in deployment.yaml`, l=label, f=follow)
kl logs -l app=<app-name>           or,     kl logs -f -l app=<app-name>
ex: kl logs -l app=fastapi-app
        INFO:     Started server process [1]
        INFO:     Waiting for application startup.
        INFO:     Application startup complete.
        INFO:     Uvicorn running on http://0.0.0.0:80 (Press CTRL+C to quit)
        INFO:     Started server process [1]
        INFO:     Waiting for application startup.
        INFO:     Application startup complete.
        INFO:     Uvicorn running on http://0.0.0.0:80 (Press CTRL+C to quit)


(after a pod is running)
----------------
kl logs -f <pod-name>
ex: kl logs -f fastapi-deployment-56bdbd747b-cmpdp
        INFO:     Started server process [1]
        INFO:     Waiting for application startup.
        INFO:     Application startup complete.
        INFO:     Uvicorn running on http://0.0.0.0:80 (Press CTRL+C to quit)


(using `--previous` flag),
ex: kl logs --previous fastapi-deployment-56bdbd747b-cmpdp
	--> Error from server (BadRequest): previous terminated container "python-fastapi-app" in pod "fastapi-deployment-56bdbd747b-cmpdp" not found

    [NOTE: This --previous flag looks for a previously crashed/restarted container. If container hasn't crashed and currently running, will throw error]



(describing entire pod)
kl describe pod <pod-name>
ex: kl describe pod fastapi-deployment-56bdbd747b-cmpdp
        Name:             fastapi-deployment-56bdbd747b-cmpdp
        Namespace:        stage-1-practicals
        Priority:         0
        Service Account:  default
        Node:             my-practicals-kind-cluster-worker2/172.18.0.4
        Start Time:       Tue, 10 Jun 2025 19:22:18 +0000
        Labels:           app=fastapi-app
                        pod-template-hash=56bdbd747b
        Annotations:      <none>
        Status:           Running
        IP:               10.244.1.4
        IPs:
        IP:           10.244.1.4



(get into KIND cluster just like `minikube ssh`):
docker ps | grep cluster-control

        --> 663d9875ecc9   kindest/node:v1.32.5   "/usr/local/bin/entr…"   2 days ago   Up 2 hours   127.0.0.1:41447->6443/tcp                                          my-practicals-kind-cluster-control-plane


then, exec into the cluster container as (this mimics `minikube ssh` command),

docker exec -it <cont-id> /bin/bash
ex: docker exec -it 663d9875ecc9 /bin/bash


then, run the curl for the  CLUSTER-IP   of the `svc`,

    root@my-practicals-kind-cluster-control-plane:/# curl 10.96.53.111
    {"message":"Hello, From DevOps Hobbies"}root@my-practicals-kind-cluster-control-plane:/# 



(BEFORE INGRESS creation - using/exposing svc to outside world via `PORT FORWARD` method),
(LOCALHOST)
# Forward local port to the service
kl port-forward svc/fastapi-service 8080:80

# Then access from another terminal or browser
curl http://localhost:8080


(EC2 REMOTE)
# Bind to all interfaces (0.0.0.0) instead of localhost, (also my 8443 was opened at inbound rule)
kl port-forward --address 0.0.0.0 svc/fastapi-service 8443:80
        Forwarding from 0.0.0.0:8443 -> 80
        Handling connection for 8443
        Handling connection for 8443


# Then access via EC2 public IP, Make sure port 8443 is open in EC2 security group
<EC2-PUBLIC-IP>:8443    (in web browser)
or, 
curl http://<EC2-PUBLIC-IP>:8443



(AFTER INGRESS creation - using/exposing svc to outside world),
kl get ing
        NAME              CLASS   HOSTS           ADDRESS   PORTS   AGE
        fastapi-ingress   nginx   fastapi.local             80      5s

        NOTE: ADDRESS is not assigned coz ingress controller is yet to be INSTALLED


## Install `nginx ingress controller`
	[NOTE (from K8 github ingress page): In AWS, we use a Network load balancer (NLB) to expose the Ingress-Nginx Controller behind a Service of Type=LoadBalancer]

kl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.3/deploy/static/provider/aws/deploy.yaml	(for `AWS kubeadm` setup)

kl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml	(for KIND clusters, creates a )

        namespace/ingress-nginx created
        serviceaccount/ingress-nginx created
        serviceaccount/ingress-nginx-admission created
        role.rbac.authorization.k8s.io/ingress-nginx created
        role.rbac.authorization.k8s.io/ingress-nginx-admission created
        clusterrole.rbac.authorization.k8s.io/ingress-nginx created
        clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
        rolebinding.rbac.authorization.k8s.io/ingress-nginx created
        rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
        clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
        clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
        configmap/ingress-nginx-controller created
        service/ingress-nginx-controller created
        service/ingress-nginx-controller-admission created
        deployment.apps/ingress-nginx-controller created
        job.batch/ingress-nginx-admission-create created
        job.batch/ingress-nginx-admission-patch created
        ingressclass.networking.k8s.io/nginx created
        validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created


then,
(see the `ingress-nginx controller` pod coming up - make sure to manually load the docker image to KIND clusters to refrain getting `ErrImageNeverPull` & `ImagePullBackOff` issues),

kl get pods -A | grep ingress-nginx-controller
        NAMESPACE              POD NAME 							  READY   STATUS    RESTARTS        AGE
        ingress-nginx          ingress-nginx-controller-684d55c96d-4mhz6                          0/1     Pending   0               110s

(should change to),
        ingress-nginx          ingress-nginx-controller-684d55c96d-mwmsc                          1/1     Running   0               69s


# Wait for it to be ready	(but make sure before each worker nodes are labelled as `ingress-ready=true`)
kl wait --namespace ingress-nginx --for=condition=ready pod --selector=app.kubernetes.io/component=controller --timeout=90s
	--> pod/ingress-nginx-controller-684d55c96d-mwmsc condition met


(check EXTERNAL-IP for the `nginx-controller` if assigned where namespace=ingress-nginx; for other setup it may be assigned. Poor KIND clusters never assigns),
kl get svc -n ingress-nginx			

        NAME                                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
        ingress-nginx-controller             NodePort    10.96.5.216    <none>        80:32735/TCP,443:30627/TCP   5m11s
        ingress-nginx-controller-admission   ClusterIP   10.96.15.164   <none>        443/TCP                      5m11s


again, check the ingress to see ADDRESS assigned   (localhost is for `KIND` clusters, for `Kubeadm` it would be NodePort IP like `192.168.x.x`  or, `10.3.x.x`, for managed distributions ex: `EKS` it would be a `A record` DNS like `k8s-game2091-ingressX-e1d9d1f206-1078327551.ap-south-1.elb.amazonaws.com`)
kl get ing
        NAME              CLASS   HOSTS           ADDRESS   PORTS   AGE
        fastapi-ingress   nginx   fastapi.local   localhost          80      5s


(Only applies to local laptop or KubeAdm setup - map ingress `hostname` to EC2 (for kubeadm worker node) or local machine /etc/hosts file),
----------
echo "<EC2-PUBLIC-IP> example.com" | sudo tee -a /etc/hosts
ex: echo "13.203.196.34 fastapi.local.org" | sudo tee -a /etc/hosts
	--> 13.203.196.34 fastapi.local.org

sudo cat /etc/hosts
        127.0.0.1 localhost
        # The following lines are desirable for IPv6 capable hosts
        ::1 ip6-localhost ip6-loopback
        fe00::0 ip6-localnet
        ff00::0 ip6-mcastprefix
        ff02::1 ip6-allnodes
        ff02::2 ip6-allrouters
        ff02::3 ip6-allhosts
        13.203.196.34 fastapi.local.org

# Remove the entry when done
sudo sed -i '/fastapi.local.org/d' /etc/hosts

## hit application from browser (for KIND it won't work)
http://fastapi.local.org/app


(for KIND setup, do below)
curl <url/endpoint> --resolve <url/endpoint>:<container-port>:<ing-ADDRESS>
ex: curl fastapi.local.org/app --resolve fastapi.local.org:80:127.0.0.1		        (this worked and printed "{"message": "Hello, From DevOps Hobbies"} coz command executed inside EC2 m/c)


(For BROWSERS, use PORT FORWARDING with INGRESS - KIND cluster is not production grade that's why)
kl port-forward --address 0.0.0.0 -n ingress-nginx svc/ingress-nginx-controller 8443:80
	Forwarding from 0.0.0.0:8443 -> 80
	Forwarding from 0.0.0.0:8443 -> 80
	Forwarding from 0.0.0.0:8443 -> 80



(Finally, CLEAN UP resources in namespace),
--------------
kl delete all --all         (This command deletes all resources (pods, services, deployments, replicasets) in the current namespace)

To be extra safe and see what will be deleted first:
kl delete all --all --dry-run=client

        --> pod "fastapi-deployment-68cb9b9b76-7vt8p" deleted (dry run)
            pod "fastapi-deployment-68cb9b9b76-gq2r6" deleted (dry run)
            deployment.apps "fastapi-deployment" deleted (dry run)
            replicaset.apps "fastapi-deployment-65cf74847f" deleted (dry run)
            replicaset.apps "fastapi-deployment-68cb9b9b76" deleted (dry run)
            replicaset.apps "fastapi-deployment-d7cdd5bbb" deleted (dry run)


Alternative - more specific:
kl delete deployment,service,pod --all

To delete everything including ingress:
kl delete all,ingress --all

        ------------------------------------------------------------------------------------------

(use `-c` flag to combine to 2 commands while exec'ng to a pod)

kl exec -ti <pod-name> -- sh -c "cd / && sh"
ex: kl exec -ti fastapi-deployment-65cf74847f-7ppv9 -- sh -c "cd / && sh"
