## [CONFIGMAPS & SECRETS - excercise]:
----------
(View Config files inside pod)
kl exec <pod-name> -- sh -c "cat <file-location-inside-pod>"

ex: kl exec fastapi-deployment-65cf74847f-7ppv9 -- sh -c "cat /etc/config/db-port"
or, kl exec fastapi-deployment-65cf74847f-7ppv9 -- cat /etc/config/db-port | more
    --> 3306


(encode base64 secret),
echo -n '<text-to-encrypt>' | base64
ex: echo -n 'rashmika-girlfr' | base64

(decode base64 secret),
echo <cipher-to-decrypt> | base64 -d        (--decode also works)
ex: echo cmFzaG1pa2EtZ2lybGZy | base64 -d


(View Secrets inside pod)
kl exec <pod-name> -- cat <secrets-file-location-inside-pod>"

ex: kl exec fastapi-deployment-68cb9b9b76-gq2r6 -- cat /mnt/secrets/db-username | more
    --> admin

kl exec fastapi-deployment-68cb9b9b76-gq2r6 -- cat /mnt/secrets/db-password | more
    --> SuperSecret123

kl exec fastapi-deployment-68cb9b9b76-gq2r6 -- cat /mnt/secrets/jwt-secret | more
    --> my_super_secret_jwt_key

kl exec fastapi-deployment-68cb9b9b76-gq2r6 -- ls -lth  /mnt/secrets
        total 0      
        lrwxrwxrwx    1 root     root          14 Jun 11 21:38 api-key -> ..data/api-key
        lrwxrwxrwx    1 root     root          14 Jun 11 21:38 db-host -> ..data/db-host
        lrwxrwxrwx    1 root     root          18 Jun 11 21:38 db-password -> ..data/db-password
        lrwxrwxrwx    1 root     root          18 Jun 11 21:38 db-username -> ..data/db-username
        lrwxrwxrwx    1 root     root          17 Jun 11 21:38 jwt-secret -> ..data/jwt-secret


        