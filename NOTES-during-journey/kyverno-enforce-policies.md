## [KYVERNO-ENFORCE-SECURITY]:
--------
1. (add helm repo for `kyverno` & update it)
hl repo add <release-name> <repo-url>
ex: hl repo add kyverno https://kyverno.github.io/kyverno/

hl repo update

2. (WARNING: always create in a specific namespace named `kyverno` NOT in working namespace)
hl install <release-name> <chart-name> -n <ns-name> --create-namespace --set replicaCount=<replica-num>

ex: hl install kyverno kyverno/kyverno -n kyverno --create-namespace --set replicaCount=1

    NAME: kyverno
    LAST DEPLOYED: Sat Jun 21 14:47:38 2025
    NAMESPACE: kyverno
    STATUS: deployed
    REVISION: 1
    NOTES:
    Chart version: 3.4.3
    Kyverno version: v1.14.3

    Thank you for installing kyverno! Your release is named kyverno.

    The following components have been installed in your cluster:
    - CRDs
    - Admission controller
    - Reports controller
    - Cleanup controller
    - Background controller

    ⚠️  WARNING: Setting the admission controller replica count below 2 means Kyverno is not running in high availability mode.

    ⚠️  WARNING: PolicyExceptions are disabled by default. To enable them, set '--enablePolicyException' to true.

    💡 Note: There is a trade-off when deciding which approach to take regarding Namespace exclusions. Please see the documentation at https://kyverno.io/docs/installation/#security-vs-operability to understand the risks.



3. (install the `kyverno specific CR` of kind: cluster policy with `Enforce` action - this WON'T DOWNLOAD the FILE locally)
curl -s https://raw.githubusercontent.com/kyverno/policies/main/best-practices/require-pod-requests-limits/require-pod-requests-limits.yaml | sed 's/validationFailureAction: Audit/validationFailureAction: Enforce/' | kubectl apply -f -

    clusterpolicy.kyverno.io/require-requests-limits configured


(see the installed CR)
4. kl get <cr-name>

ex: kl get clusterpolicy

    NAME                      ADMISSION   BACKGROUND   READY   AGE     MESSAGE
    require-requests-limits   true        true         True    8m45s   Ready


(or, detailed one),
kl get clusterpolicy require-requests-limits -o yaml   | grep validationFailureAction: 

    --> validationFailureAction: Enforce


5. (then deploy the nginx application without **resource requests & limits**),
kl apply -f deployment.yaml          (use same file content [deployment-nginx.yaml](../stage-1-practicals/deployment-nginx.yaml) ---- kyverno should throw error)

    Error from server: error when creating "deployment.yaml": admission webhook "validate.kyverno.svc-fail" denied the request: 

    resource Deployment/enforce-security/nginx-deployment was blocked due to the following policies 

    require-requests-limits:
    autogen-validate-resources: 'validation error: CPU and memory resource requests
        and memory limits are required for containers. rule autogen-validate-resources
        failed at path /spec/template/spec/containers/0/resources/limits/'


(see all pods running in `kyverno` namespace after `ClusterPolicy` injected)
kl get pods -A | grep <ns-name>

ex: kl get pods -A | grep kyverno

    kyverno                kyverno-admission-controller-846966d4d6-rxbh2                      1/1     Running   0              11m
    kyverno                kyverno-background-controller-74f47ccf5c-p9glp                     1/1     Running   0              11m
    kyverno                kyverno-cleanup-controller-7b5c94fcbd-l6z8b                        1/1     Running   0              11m
    kyverno                kyverno-reports-controller-675f66f98-jslpq                         1/1     Running   0              11m



6. hl uninstall <release-name> -n <ns-name>
ex: hl uninstall kyverno -n kyverno-example

    These resources were kept due to the resource policy:
    [ConfigMap] kyverno

    release "kyverno" uninstalled


