## [TAINTS-TOLERATIONS]:
--------
1. edit control-plane and add a node-selector label (optional)
kl edit node <control-plane-name>
ex: kl edit node my-practicals-kind-cluster-control-plane

    labels:
    beta.kubernetes.io/arch: amd64
    beta.kubernetes.io/os: linux
    kubernetes.io/arch: amd64
    kubernetes.io/hostname: my-practicals-kind-cluster-control-plane
    kubernetes.io/os: linux
    node-role.kubernetes.io/control-plane: ""
    node-name: control-plane-only                           # add this label (optional)
    node.kubernetes.io/exclude-from-external-load-balancers: ""
    name: my-practicals-kind-cluster-control-plane


2. Untaint Control Plane (Remove default taint):
# Remove the default NoSchedule taint from control plane
kl taint nodes <control-plane-name> node-role.kubernetes.io/control-plane:NoSchedule-

# For Kind clusters specifically
ex: kl taint nodes my-practicals-kind-cluster-control-plane node-role.kubernetes.io/control-plane:NoSchedule-

# Check if taint is removed
kl describe node <control-plane-name> | grep Taints
ex: kl describe node my-practicals-kind-cluster-control-plane | grep Taints

        Taints:             node-role.kubernetes.io/control-plane:NoSchedule            (this o/p shown if Taint is re-applied)


3. apply the `stage-1-practicals/taints-tolerations/deployment-nginx.yaml` via
kl apply -f deployment-nginx.yaml


4. delete deployment and reapply the taint
kl taint nodes my-practicals-kind-cluster-control-plane node-role.kubernetes.io/control-plane:NoSchedule


