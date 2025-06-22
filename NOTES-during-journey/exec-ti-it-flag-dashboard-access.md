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

eyJhbGciOiJSUzI1NiIsImtpZCI6IkMxWHZoQlJfV2lfTE1FU0hpZHhmTlRvZFpQSlRzZzA4R0RYSXBGZGlWMXMifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzQ5NDA0Mzg5LCJpYXQiOjE3NDk0MDA3ODksImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwianRpIjoiMmFhYTY0MzgtYmM5NC00NmIwLTgwMjQtM2Q2MmY1MTAzN2M0Iiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJhZG1pbi11c2VyIiwidWlkIjoiY2M2NGM1ZGYtZjlkZi00MmUxLThhNGMtNjJiNTVjMmQ4NzRhIn19LCJuYmYiOjE3NDk0MDA3ODksInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDphZG1pbi11c2VyIn0.Yc2vZqjYyi3SMdgFiJRLr_4ApDa-k0w4S11hIIIoSOp0KPVuR9Ez89OPGKiovlg6IMPKlzmYo3isgzFx49ZrfueL1rz26Q8jpe87mtjF6pt9oxqfecV96iRaXaXD-IM2UjphdWrIUlUnCAjOWgBABfBkD4or7Wnb0kQGtUyEmPXd3IBd1K1vibtK_rbhz61e0OLek3c6TEKjGYwu5NA3ChTJVf0DvArKj-9eSURr_1hbaOVSzqmBBeZtzdyZJd2GQOoOedsRoePY14IzkkAK4ANBAoFyh25UI0tcdyq77C5of9yqs13tkoIgihkVEHVe0Kh5JktIPb5IwsJ3pDxbdA


# Access via: 
https://<EC2-PUBLIC-IP>:8443

sudo lsof -i :8001
sudo kill <PID>



