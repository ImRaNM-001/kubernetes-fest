## [RBAC - Role, RoleBindings, ClusterRole, ClusterRoleBindings]:
--------
1> (create separate namespaces 1st)
kl create ns dev-team                        [IMPERATIVE WAY]

or,
kl apply -f namespaces.yaml                 [DECLARATIVE WAY]

    --> Warning: resource namespaces/dev-team is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
    namespace/dev-team configured
    namespace/qa-team created
    namespace/ops-team created
(I go this WARNING coz `dev-team` ns was already created imperative way)


2> create `role` & `rolebinding`,
kl apply -f role.yaml
kl apply -f rolebinding.yaml

or, series of them,
kl apply -f clusterrole-ops-team.yaml -f clusterrolebinding-ops-team.yaml -f role-dev-team.yaml -f role-qa-team.yaml -f role.yaml -f rolebinding-dev-team.yaml -f rolebinding-qa-team.yaml -f rolebinding.yaml


3> (see role & rolebinding for `dev` & `qa` team together)
kl get role,rolebinding -A | grep team

        dev-team               role.rbac.authorization.k8s.io/dev-role                                         2025-06-13T08:16:59Z
        dev-team               role.rbac.authorization.k8s.io/dev-team-role                                    2025-06-13T08:47:03Z
        qa-team                role.rbac.authorization.k8s.io/qa-team-role                                     2025-06-13T08:47:03Z
        dev-team               rolebinding.rbac.authorization.k8s.io/dev-role-binding                                    Role/dev-role                                         33m
        dev-team               rolebinding.rbac.authorization.k8s.io/dev-team-rolebinding                                Role/dev-team-role                                    4m24s
        qa-team                rolebinding.rbac.authorization.k8s.io/qa-team-rolebinding                                 Role/qa-team-role  


4> (see clusterole & clusterrolebinding for `ops` team)
kl get clusterrole,clusterrolebinding | grep ops

        clusterrole.rbac.authorization.k8s.io/ops-team-role                                                          2025-06-13T08:47:03Z
        clusterrolebinding.rbac.authorization.k8s.io/ops-team-rolebinding                                            ClusterRole/ops-team-role                                                          6m26s


5> Now, Verify RBAC Configuration,

(a) Option 1: User impersonation (EASIEST & SIMPLEST) : this is when `rolebinding` is using `Group` and not `ServiceAccount`,
# Test as QA user without creating actual user
kl auth can-i delete pods -n qa-team --as=qa-user --as-group=qa-team@example.com
    --> no

kl auth can-i get pods -n qa-team --as=qa-user --as-group=qa-team@example.com
    --> yes


(b) Option 2: Update qa-team's RoleBinding to use `ServiceAccount` instead of `Group` --> [see the yaml manifests saved]

In KIND cluster, the cluster-admin privileges (admin context) are NEVER overridden with the `token` flag, hence:
--as impersonation method works correctly:
--as=system:serviceaccount:qa-team:qa-team-svc-accnt

This actually tests as the ServiceAccount Bypasses the admin context.


# Tested as the service account directly
kl auth can-i create pods -n qa-team --as=system:serviceaccount:qa-team:qa-team-svc-accnt
        --> no

kl auth can-i get pods -n qa-team --as=system:serviceaccount:qa-team:qa-team-svc-accnt
        --> yes

kl auth can-i delete pods -n qa-team --as=system:serviceaccount:qa-team:qa-team-svc-accnt
        --> no

kl auth can-i create services -n qa-team --as=system:serviceaccount:qa-team:qa-team-svc-accnt
        --> no

kl auth can-i list deployments -n qa-team --as=system:serviceaccount:qa-team:qa-team-svc-accnt
        --> yes
        

also, we can double check what permissions the `qa-team` actually has via,
kl describe role qa-team-role -n qa-team

        Name:         qa-team-role
        Labels:       <none>
        Annotations:  <none>
        PolicyRule:
        Resources    Non-Resource URLs  Resource Names  Verbs
        ---------    -----------------  --------------  -----
        deployments  []                 []              [get list]
        pods         []                 []              [get list]
        services     []                 []              [get list]



(c) Else (for Cloud Vendors - EKS, AKS) - Switch to a QA Team user and check access:
NOTE: Coz creating a QA user context in KIND cluster is complex. Few steps experimented:

firstly, check current user as,
kl auth whoami

        ATTRIBUTE                                           VALUE
        Username                                            kubernetes-admin
        Groups                                              [kubeadm:cluster-admins system:authenticated]
        Extra: authentication.kubernetes.io/credential-id   [X509SHA256=2130cc7ed656c49c09ef7f60be41dd89080b95c509aa924f56b06acc0abcdb1e]

and check `current-context` as,
kl config current-context
        
        --> kind-my-practicals-kind-cluster

    This Cluster admin overrides everything, using `kind-my-practicals-kind-cluster` context means you have full admin access.
    Admin can do anything regardless of roles. 
    To properly test QA team access:

------------------------------------------------
[ALTERNATIVE 1]: Create a QA user certificate/token,

QA_TOKEN=$(kl create token <svc-accnt-name> -n <ns-name>)
ex: QA_TOKEN=$(kl create token qa-team-svc-accnt -n qa-team)

Then, hit below commands for test:

# Test allowed verbs (should return "yes"):

    kl auth can-i get pods -n qa-team --token=$QA_TOKEN
    kl auth can-i list pods -n qa-team --token=$QA_TOKEN
    kl auth can-i get services -n qa-team --token=$QA_TOKEN
    kl auth can-i list services -n qa-team --token=$QA_TOKEN
    kl auth can-i get deployments -n qa-team --token=$QA_TOKEN
    kl auth can-i list deployments -n qa-team --token=$QA_TOKEN

# Test forbidden verbs (should return "no"):

    kl auth can-i create pods -n qa-team --token=$QA_TOKEN
    kl auth can-i delete pods -n qa-team --token=$QA_TOKEN
    kl auth can-i update pods -n qa-team --token=$QA_TOKEN
    kl auth can-i patch services -n qa-team --token=$QA_TOKEN
    kl auth can-i delete deployments -n qa-team --token=$QA_TOKEN

# Test other namespaces (should return "no"):

    kl auth can-i get pods -n default --token=$QA_TOKEN
    kl auth can-i get pods -n dev-team --token=$QA_TOKEN
    kl auth can-i '*' pods -n qa-team --token=$QA_TOKEN                     # Should be "no"



------------------------------------------------
[ALTERNATIVE 2]: Switch to QA user context:

kl config use-context qa-user-context                    # Not admin context - KIND clusters FAILS AT THIS STEP
        --> error: no context exists with the name: "qa-user-context"

then test,

    kl auth can-i delete pods -n qa-team  # Should be "no"
    kl auth can-i get pods -n qa-team     # Should be "yes"



    