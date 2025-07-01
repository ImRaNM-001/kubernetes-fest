## [POD-PRIORITY-PREEMPTION]:
--------
1. Create 3 priority classes with different priority values

2. Deploy these pods with different priority classes

3. Create a resource-constrained deployment with 5 replicas and resource requests set

4. Check your node resources

5. Deploy a critical-priority pod with high resource utilization and observe preemption

---

(`priority classes` available by default)
```bash
kl get pc

    NAME                      VALUE        GLOBAL-DEFAULT   AGE   PREEMPTIONPOLICY
    system-cluster-critical   2000000000   false            22d   PreemptLowerPriority
    system-node-critical      2000001000   false            22d   PreemptLowerPriority
```

(view structure of `priority class` template)
```bash
kl explain pc

    GROUP:      scheduling.k8s.io
    KIND:       PriorityClass
    VERSION:    v1

    DESCRIPTION:
        PriorityClass defines mapping from a priority class name to the priority
        integer value. The value can be any valid integer.
        
    FIELDS:
    apiVersion	<string>
        APIVersion defines the versioned schema of this representation of an object.
        Servers should convert recognized schemas to the latest internal value, and
        may reject unrecognized values. More info:
        https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

    description	<string>
        description is an arbitrary string that usually provides guidelines on when
        this priority class should be used.

    globalDefault	<boolean>
        globalDefault specifies whether this PriorityClass should be considered as
        the default priority for pods that do not have any priority class. Only one
        PriorityClass can be marked as `globalDefault`. However, if more than one
        PriorityClasses exists with their `globalDefault` field set to true, the
        smallest value of such global default PriorityClasses will be used as the
        default priority.

    kind	<string>
        Kind is a string value representing the REST resource this object
        represents. Servers may infer this from the endpoint the client submits
        requests to. Cannot be updated. In CamelCase. More info:
        https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

    metadata	<ObjectMeta>
        Standard object's metadata. More info:
        https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

    preemptionPolicy	<string>
    enum: Never, PreemptLowerPriority
        preemptionPolicy is the Policy for preempting pods with lower priority. One
        of Never, PreemptLowerPriority. Defaults to PreemptLowerPriority if unset.
        
        Possible enum values:
        - `"Never"` means that pod never preempts other pods with lower priority.
        - `"PreemptLowerPriority"` means that pod can preempt other pods with lower
        priority.

    value	<integer> -required-
        value represents the integer value of this priority class. This is the
        actual priority that pods receive when they have the name of this class in
        their pod spec.
```

## Task 1: 
(apply the `priority classes`:
[high-priority-class](../stage-2-practicals/PodPriority-Preemption/priority-classes-manifests/high-priority-class.yaml), 
[medium-priority-class](../stage-2-practicals/PodPriority-Preemption/priority-classes-manifests/medium-priority-class.yaml), 
[low-priority-class](../stage-2-practicals/PodPriority-Preemption/priority-classes-manifests/low-priority-class.yaml) and check)

```bash
kl get pc -o wide

    NAME                      VALUE        GLOBAL-DEFAULT   AGE     PREEMPTIONPOLICY
    high-priority             1000000      false            2m40s   PreemptLowerPriority
    low-priority              10000        true             2m40s   PreemptLowerPriority
    medium-priority           100000       false            2m40s   PreemptLowerPriority
    system-cluster-critical   2000000000   false            22d     PreemptLowerPriority
    system-node-critical      2000001000   false            22d     PreemptLowerPriority
```

## Task 2, 3 & 4:
(apply the different priority pods including the 5 replica [resource-hog](../stage-2-practicals/PodPriority-Preemption/resource-hog.yaml) (low priority) pods)

```bash
kl get pods -w

    NAME                            READY   STATUS              RESTARTS   AGE
    high-priority-nginx             0/1     ContainerCreating   0          7s
    low-priority-nginx              0/1     ContainerCreating   0          7s
    medium-priority-nginx           0/1     ContainerCreating   0          7s
    resource-hog-64fd7ff65f-72flw   0/1     ContainerCreating   0          7s
    resource-hog-64fd7ff65f-d8x88   0/1     Pending             0          7s
    resource-hog-64fd7ff65f-gwdvh   0/1     ContainerCreating   0          7s
    resource-hog-64fd7ff65f-jglt5   0/1     ContainerCreating   0          7s
    resource-hog-64fd7ff65f-m6w67   0/1     ContainerCreating   0          7s
    low-priority-nginx              1/1     Running             0          11s
    high-priority-nginx             1/1     Running             0          11s
    resource-hog-64fd7ff65f-jglt5   1/1     Running             0          12s
    medium-priority-nginx           1/1     Running             0          13s
    resource-hog-64fd7ff65f-m6w67   1/1     Running             0          14s
    resource-hog-64fd7ff65f-gwdvh   1/1     Running             0          14s
    resource-hog-64fd7ff65f-72flw   1/1     Running             0          16s
```

## Task 5:
(next apply the `critical pod` and check the preemption)

kl apply -f [critical-priority-pod](../stage-2-practicals/PodPriority-Preemption/critical-priority-pod.yaml).yaml

```bash
    pod/critical-priority-nginx created
```

(low priority pods are evicted/preempted/kicked out)

```bash
kl get pods -w

    NAME                            READY   STATUS              RESTARTS   AGE
    critical-priority-nginx         0/1     ContainerCreating   0          4s
    low-priority-nginx              0/1     Pending             0          25s
    medium-priority-nginx           1/1     Running             0          49s
    resource-hog-64fd7ff65f-27k75   0/1     Pending             0          3s
    resource-hog-64fd7ff65f-9zdfh   1/1     Running             0          2m58s
    resource-hog-64fd7ff65f-dxm4t   1/1     Running             0          2m59s
    resource-hog-64fd7ff65f-gccpm   0/1     Pending             0          3s
    resource-hog-64fd7ff65f-vl968   0/1     Pending             0          3s
    critical-priority-nginx         1/1     Running             0          5s
```

(view preemption info from `events` of kube-API-server-collected from kubelet, controllers)

```bash
kl get events | grep -i "preempted by pod"

    3m27s       Normal    Preempted           pod/low-priority-nginx               Preempted by pod e332ef51-eda3-4c25-a18a-eb053212a21c on node my-practicals-kind-cluster-worker
    17m         Normal    Preempted           pod/resource-hog-64fd7ff65f-72flw    Preempted by pod bfa918d8-3b1f-4343-a2a3-208304c11d3b on node my-practicals-kind-cluster-worker2
    15m         Normal    Preempted           pod/resource-hog-64fd7ff65f-d8x88    Preempted by pod 5441cad3-ec18-4fcd-8376-0e6a31763bc3 on node my-practicals-kind-cluster-worker
    2m41s       Normal    Preempted           pod/resource-hog-64fd7ff65f-f2scj    Preempted by pod 68dc9c7b-c8f3-42f7-88df-c17c41504d52 on node my-practicals-kind-cluster-worker2
    5m37s       Normal    Preempted           pod/resource-hog-64fd7ff65f-fb2vs    Preempted by pod a7dcb678-be26-4c07-be46-a64009153b8c on node my-practicals-kind-cluster-worker
    2m41s       Normal    Preempted           pod/resource-hog-64fd7ff65f-g8f78    Preempted by pod 68dc9c7b-c8f3-42f7-88df-c17c41504d52 on node my-practicals-kind-cluster-worker2
    2m41s       Normal    Preempted           pod/resource-hog-64fd7ff65f-jkmhw    Preempted by pod 68dc9c7b-c8f3-42f7-88df-c17c41504d52 on node my-practicals-kind-cluster-worker2
    17m         Normal    Preempted           pod/resource-hog-64fd7ff65f-m6w67    Preempted by pod bfa918d8-3b1f-4343-a2a3-208304c11d3b on node my-practicals-kind-cluster-worker2
    5m37s       Normal    Preempted           pod/resource-hog-64fd7ff65f-vlwvm    Preempted by pod a7dcb678-be26-4c07-be46-a64009153b8c on node my-practicals-kind-cluster-worker
```


(Finally, CLEAN UP),
--------------
(delete all resources in current/working namespace)

```bash
kl delete all --all -n <ns-name>

ex: kl delete all --all -n examples

    pod "critical-priority-nginx" deleted
    pod "low-priority-nginx" deleted
    pod "medium-priority-nginx" deleted
    pod "resource-hog-64fd7ff65f-27k75" deleted
    pod "resource-hog-64fd7ff65f-9zdfh" deleted
    pod "resource-hog-64fd7ff65f-dxm4t" deleted
    pod "resource-hog-64fd7ff65f-gccpm" deleted
    pod "resource-hog-64fd7ff65f-vl968" deleted
    deployment.apps "resource-hog" deleted
    replicaset.apps "resource-hog-64fd7ff65f" deleted
```

(delete the `priority classes`)

```bash
kl delete pc <pc1> <pc2> <pc3>

ex: kl delete pc high-priority low-priority medium-priority

    priorityclass.scheduling.k8s.io "high-priority" deleted
    priorityclass.scheduling.k8s.io "low-priority" deleted
    priorityclass.scheduling.k8s.io "medium-priority" deleted
```