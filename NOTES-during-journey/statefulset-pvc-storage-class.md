## [STATEFULSETS-PVC-STORAGECLASS]:
--------
(encode password to base64 for storing in `secrets.yaml`)
echo -n "root" | base64
cm9vdA==


(create all the yaml manifests)
kl apply -f .

if below error shows - no worries,

        Error from server (NotFound): error when creating "configMap.yaml": namespaces "mysql-db" not found

(DO again)     kl apply -f .



(see statefulset & svc inside the `mysql-db` ns)
kl get all -n <ns-name>
ex: kl get all -n mysql-db		 

        NAME                TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
        service/mysql-svc   ClusterIP   None         <none>        3306/TCP   14m

        NAME                                 READY   AGE
        statefulset.apps/mysql-statefulset   0/3     14m


(watch pods in the `mysql-db` ns)
kl get pods -n <ns-name> -w	
ex: kl get pods -n mysql-db -w		

        NAME                  READY   STATUS              RESTARTS   AGE
        mysql-statefulset-0   0/1     ContainerCreating   0          19s
        mysql-statefulset-0   1/1     Running             0          22s
        mysql-statefulset-1   0/1     Pending             0          0s
        mysql-statefulset-0   0/1     Error               0          24s
        mysql-statefulset-0   1/1     Running             1 (1s ago)   25s


(to debug statefulset)
kl describe pod <pod-name> -n <ns-name>
ex: kl describe pod mysql-statefulset-0 -n mysql-db		

        Events:
        Type     Reason     Age                   From               Message
        ----     ------     ----                  ----               -------
        Normal   Scheduled  2m24s                 default-scheduler  Successfully assigned mysql-db/mysql-statefulset-1 to my-practicals-kind-cluster-worker2
        Normal   Pulled     53s (x5 over 2m24s)   kubelet            Container image "mysql:8.0" already present on machine
        Normal   Created    53s (x5 over 2m24s)   kubelet            Created container: mysql
        Normal   Started    53s (x5 over 2m23s)   kubelet            Started container mysql
        Warning  BackOff    13s (x10 over 2m17s)  kubelet            Back-off restarting failed container mysql in pod mysql-statefulset-1_mysql-db(535c1b79-701d-46a0-adca-636cb8b0b559)


(check logs of one of the pod `mysql-statefulset-1`)
kl logs <pod-name> -n <ns-name>
ex: kl logs mysql-statefulset-1 -n mysql-db

        2025-06-13 18:32:20+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.42-1.el9 started.
        2025-06-13 18:32:21+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
        2025-06-13 18:32:21+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.42-1.el9 started.
        2025-06-13 18:32:22+00:00 [ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
        You need to specify one of the following as an environment variable:
        - MYSQL_ROOT_PASSWORD
        - MYSQL_ALLOW_EMPTY_PASSWORD
        - MYSQL_RANDOM_ROOT_PASSWORD

MISTAKE 1: messed up with MYSQL default expected env vars
MISTAKE 2: not given sufficient memory (limits) to pods in statefulset

(to debug statefulset)
kl describe statefulset <statefulset-name> -n <ns-name>
ex: kl describe statefulset mysql-statefulset -n mysql-db		


(increased resource "limits" quota to overcome OOMKill error), then
kl get pods -n <ns-name> -w			
ex: kl get pods -n mysql-db -w			

        NAME                  READY   STATUS    RESTARTS   AGE
        mysql-statefulset-0   1/1     Running   0          9s
        mysql-statefulset-1   1/1     Running   0          7s
        mysql-statefulset-2   1/1     Running   0          6s


(finally, exec'ng to pods)
kl exec -it <pod-name> -n <ns-name> -- sh
ex: kl exec -it mysql-statefulset-2 -n mysql-db -- sh

        --> sh-5.1# mysql -u root -p
        
        --> Enter password:                     (type root and press enter)
        
        Welcome to the MySQL monitor.  Commands end with ; or \g.
        Your MySQL connection id is 8
        Server version: 8.0.42 MySQL Community Server - GPL

        Copyright (c) 2000, 2025, Oracle and/or its affiliates.

        Oracle is a registered trademark of Oracle Corporation and/or its
        affiliates. Other names may be trademarks of their respective
        owners.

        Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

        mysql> show databases;
        +--------------------+
        | Database           |
        +--------------------+
        | devops-db          |
        | information_schema |
        | mysql              |
        | performance_schema |
        | sys                |
        +--------------------+
        5 rows in set (0.00 sec)

or, directly
kl exec -it mysql-statefulset-0 -n mysql-db -- mysql -u root -p


-----------------------------------------------------------------------------
[VERY IMPORTANT]: ALWAYS delete `PVC` BEFORE deleting `statefulset` coz the junk/dirty data always gets bloody persisted.

Detailed debugging steps:
----
# Fix and reapplied the corrected secret with `type: Opaque`
kl apply -f secrets.yaml

# Restart StatefulSet to pick up changes                [VERY USEFUL]
kl rollout restart statefulset <statefulset-name> -n <ns-name>
ex: kl rollout restart statefulset mysql-statefulset -n mysql-db

        statefulset.apps/mysql-statefulset restarted

# Wait for pods to be ready
kl wait --for=condition=ready pod -l app=<app-name> -n <ns-name> --timeout=60s
ex: kl wait --for=condition=ready pod -l app=mysql -n mysql-db --timeout=60s

        pod/mysql-statefulset-0 condition met
        pod/mysql-statefulset-1 condition met
        pod/mysql-statefulset-2 condition met



The password is still not working. Let's debug step by step:
## **1. Check if secret is properly applied:**
```bash
# Verify secret content
kl get secret mysql-db-secret -n mysql-db -o yaml

# Decode the password to verify
kl get secret mysql-db-secret -n mysql-db -o jsonpath='{.data.MYSQL_ROOT_PASSWORD}' | base64 -d
```
                root

## **2. Check pod environment variables:**
```bash
kl exec mysql-statefulset-0 -n mysql-db -- env | grep MYSQL
```
        MYSQL_MAJOR=8.0
        MYSQL_VERSION=8.0.42-1.el9
        MYSQL_SHELL_VERSION=8.0.42-1.el9
        MYSQL_ROOT_PASSWORD=root
        MYSQL_DATABASE=devops-db

## **3. Check full MySQL logs:**
```bash
kl logs mysql-statefulset-0 -n mysql-db
```

## **4. Try connecting without password first:**
```bash
# Check if MySQL started with empty password due to config issues
kl exec -it mysql-statefulset-0 -n mysql-db -- mysql -u root
```

## **5. If still failing, recreate everything:**
```bash
# Delete StatefulSet and let it recreate with fresh data
kl delete statefulset mysql-statefulset -n mysql-db
kl delete pvc -l app=mysql -n mysql-db                          # Delete persistent data [MAJOR CULPRIT]
kl apply -f statefulset.yaml
```


The issue is clear now! Everything is configured correctly, but there are two problems:

## **1. Password authentication issue:**
MySQL is running but there's an authentication problem. The environment variables are set correctly, but MySQL might not have the password configured properly during initialization.

## **2. Missing database:**
The `devops-db` database wasn't created even though `MYSQL_DATABASE=devops-db` is set.

## **Solution - Fresh start with clean data:**

```bash
# Exit MySQL first
mysql> exit

# Delete StatefulSet and PVCs to start fresh
kl delete statefulset mysql-statefulset -n mysql-db
kl delete pvc -l app=mysql -n mysql-db                         # [This was the bloody MAJOR CULPRIT]

# Wait for deletion
kl get pvc -n mysql-db

# Recreate StatefulSet
kl apply -f statefulset.yaml

# Wait for pods to be ready
kl wait --for=condition=ready pod mysql-statefulset-0 -n mysql-db --timeout=120s
```

## **Test after recreation:**
```bash
# Check logs to ensure proper initialization
kl logs mysql-statefulset-0 -n mysql-db | grep -i "database\|ready"

# Test connection with password
kl exec -it mysql-statefulset-0 -n mysql-db -- mysql -u root -p
# Enter password: root

# Check if devops-db exists
mysql> SHOW DATABASES;
```
