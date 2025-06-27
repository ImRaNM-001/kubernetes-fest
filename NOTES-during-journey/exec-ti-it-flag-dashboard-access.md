        --------------------------------------OTHER THINGS----------------------------------------------------
(getting inside pod - using `exec` - various flags)

kl exec --stdin --tty <pod-name> -- /bin/bash		
ex: kl exec --stdin --tty fastapi-deployment-56bdbd747b-cmpdp -- /bin/sh	(I'd put "sh" coz of the alpine dockerimage - no bash was there)
    /app $ pwd
    /app

[NOTE: The --tty (-t) flag allocates a TTY (terminal) so you get an interactive shell experience — like a real terminal.
Without --tty, things like clear screen, arrow keys, or colors may not work properly inside the pod shell. It's what makes the session "feel" like a real terminal]


(EXACTLY EQUIVALENT TO)
kl exec -ti nginx-app-5jyvm -- /bin/sh			
ex:  kl exec -ti fastapi-deployment-56bdbd747b-cmpdp -- /bin/sh		(same o/p as above)


**`-it` and `-ti` are exactly the same!**

- **`-i`** = `--stdin` (Keep stdin open)
- **`-t`** = `--tty` (Allocate a pseudo-TTY)
- **Order doesn't matter**: `-it` = `-ti`

## **Flag breakdown:**
```bash
kubectl exec -i -t pod-name -- sh    # Separate flags
kubectl exec -it pod-name -- sh      # Combined flags
kubectl exec -ti pod-name -- sh      # Same as above, different order
```
**All three commands above are identical in functionality.**


===========================================================================================
## [K8 DASHBOARD ACCESS]:
--------
after applying the conf "kl apply -f dashboard-admin-user.yaml" from shubham's page, then

kl get svc -n kubernetes-dashboard           (ns = kubernetes-dashboard)

    NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    dashboard-metrics-scraper   ClusterIP   10.96.38.226    <none>        8000/TCP   50m
    kubernetes-dashboard        ClusterIP   10.96.235.250   <none>        443/TCP    50m


# Bind to all interfaces for external access
kl port-forward -n kubernetes-dashboard --address 0.0.0.0 svc/kubernetes-dashboard 8443:443


(create token)
kl -n kubernetes-dashboard create token admin-user

    ey..


# Access via: 
https://<EC2-PUBLIC-IP>:8443

sudo lsof -i :8001
sudo kill <PID>



