## [HEADLESS-SVC-STATEFULSETS]:
--------
follow `NOTES-during-journey/statefulset-pvc-storage-class.md`
for creation of required manifests/apps - 2nd pt.               (use same yaml manifests)


(then see all **k8 objects** running in the appropriate namespace),
kl get all -n <ns-name>

ex: kl get all -n mysql-db 

        NAME                      READY   STATUS    RESTARTS   AGE
        pod/mysql-statefulset-0   1/1     Running   0          4m47s
        pod/mysql-statefulset-1   1/1     Running   0          4m7s
        pod/mysql-statefulset-2   1/1     Running   0          4m3s

        NAME                TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
        service/mysql-svc   ClusterIP   None         <none>        3306/TCP   4m47s

        NAME                                 READY   AGE
        statefulset.apps/mysql-statefulset   3/3     4m47s



## Test StatefulSet DNS resolution:

### (a) Test DNS resolution for StatefulSet pod,
kl run <test-temp-pod-name> --image=<dummyBackend-image-name> --rm -it --restart=Never -n <ns-name> -- nslookup <statefulset-pod-name>.<headless-svc-name>.<ns-name>.svc.cluster.local

ex: kl run dns-test --image=busybox --rm -it --restart=Never -n mysql-db -- nslookup mysql-statefulset-1.mysql-svc.mysql-db.svc.cluster.local

                Server:		10.96.0.10
                Address:	10.96.0.10:53

                Name:	mysql-statefulset-1.mysql-svc.mysql-db.svc.cluster.local
                Address: 10.244.2.5


                pod "dns-test" deleted


### (b) Test the service DNS,       (here `statefulset pod name` is removed thereby fetching DNS for all 3 statefulset pods)
kl run <test-temp-pod-name> --image=<dummyBackend-image-name> --rm -it --restart=Never -n <ns-name> -- nslookup <headless-svc-name>.<ns-name>.svc.cluster.local

ex: kl run dns-test --image=busybox --rm -it --restart=Never -n mysql-db -- nslookup mysql-svc.mysql-db.svc.cluster.local

                Server:		10.96.0.10
                Address:	10.96.0.10:53

                Name:	mysql-svc.mysql-db.svc.cluster.local
                Address: 10.244.1.5
                Name:	mysql-svc.mysql-db.svc.cluster.local
                Address: 10.244.2.5
                Name:	mysql-svc.mysql-db.svc.cluster.local
                Address: 10.244.1.7


                pod "dns-test" deleted



## (c) (Interactive DNS testing) - Started busybox pod for interactive testing,
kl run dns-test --image=busybox --rm -it --restart=Never -n mysql-db -- sh

ex: kl run dns-test --image=busybox --rm -it --restart=Never -n mysql-db -- sh

        If you don't see a command prompt, try pressing enter.

        / # nslookup mysql-statefulset-0.mysql-svc.mysql-db.svc.cluster.local                 (1st testing for `statefulset pod 0`)
        Server:		10.96.0.10
        Address:	10.96.0.10:53


                Name:	mysql-statefulset-0.mysql-svc.mysql-db.svc.cluster.local                        
                Address: 10.244.1.5

        / #  nslookup mysql-svc.mysql-db.svc.cluster.local                       (2nd testing for all 3 `statefulset pods` via the headless svc)
        Server:		10.96.0.10
        Address:	10.96.0.10:53


                Name:	mysql-svc.mysql-db.svc.cluster.local
                Address: 10.244.1.5
                Name:	mysql-svc.mysql-db.svc.cluster.local
                Address: 10.244.1.7
                Name:	mysql-svc.mysql-db.svc.cluster.local
                Address: 10.244.2.5




