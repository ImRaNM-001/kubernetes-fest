## Drawbacks of KIND (Kubernetes IN Docker)
--------
Local, lightweight dev environment

✅ Runs K8s clusters in Docker containers for local dev/testing

✅ Great for CI/testing

❌ No real cloud integrations (IAM, ELB, VPC, etc.)

❌ No auto-scaling, node groups, or managed control plane

❌ Not suitable for production workloads

❌ Limited networking and load balancer support


In KIND, it's just one EC2 instance (or local machine) running Docker containers for control plane and nodes.

✅ No EBS volumes, Elastic IPs, or cloud networking
✅ All containers can be removed anytime
✅ No background cloud costs once EC2 is stopped or terminated



EKS (Elastic Kubernetes Service)
--------
Fully managed, production-ready K8s by AWS

✅ Managed control plane

✅ Integrated with AWS services (IAM, ALB, CloudWatch, etc.)

✅ Auto-scaling, node groups, Fargate support

✅ High availability & multi-AZ

✅ Scalable, secure production deployments in AWS.

❌ More complex & costly for small testing


Q1: can one eks cluster have 3 master nodes and 100 worker nodes ?
Ans: No — in EKS, the control plane (masters) is fully managed by AWS and is not user-configurable (you can’t set 3 masters manually).

But yes, you can have 100+ worker nodes (EC2 or Fargate) in a single EKS cluster


Q2: then what about scaling control plane and high availability setup? should I create more clusters in different A-Z's liker americas, europe and asia to get 3 master nodes?

Ans: Control Plane Scaling & High Availability in EKS:
**EKS automatically provisions a multi-AZ control plane (3 master nodes across 3 Availability Zones)** by default, in the region you select.

You don’t need to create or manage these master nodes — AWS handles HA, scaling, failover, and backups for you.


What about global HA (America, Europe, Asia)?
For regional resilience, you need to create separate EKS clusters in each region (e.g., us-west-2, eu-central-1, ap-southeast-1).

Then use multi-cluster strategies or tools like Cluster API, Rancher, Submariner, or Service Mesh (Istio/Linkerd) to connect or failover between them.

So in short:

✅ 3-master HA is built-in within a single EKS region

🌐 Cross-region HA = multiple clusters + cross-cluster setup