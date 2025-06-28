(install eksctl & check version at 1 go)
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

    0.210.0


(attach new policy `EKSDescribeClusterVersionsPolicy` as root user in the ui console)               [TRIED & WORKED]

save this in current working directory (of instance) as [eks-describe-versions-policy.json](../stage-4-practicals/eks-custom-policies/eks-describe-versions-policy.json)      (later useful for imperative execution)        


(verify if IAM user can do `describe clusters`)
aws eks describe-cluster-versions 

    {
    "clusterVersions": [
        {
            "clusterVersion": "1.33",
            "clusterType": "eks",
            "defaultPlatformVersion": "eks.6",
            "defaultVersion": true,
            "releaseDate": "2025-05-29T00:00:00+00:00",
            "endOfStandardSupportDate": "2026-07-29T00:00:00+00:00",
            "endOfExtendedSupportDate": "2027-07-29T00:00:00+00:00",
            "status": "STANDARD_SUPPORT",
            "versionStatus": "STANDARD_SUPPORT",
            "kubernetesPatchVersion": "1.33.1"
        },



(then add below permissions)
- AWSCloudFormationFullAccess
- AmazonEKSClusterPolicy
- AmazonEKSServicePolicy
- AmazonEKSWorkerNodePolicy
- IAMFullAccess                     (needed for quick adding policies/roles; revoke once project done)


(if below error while creating clusters)

    2025-06-28 11:12:15 [ℹ]  building cluster stack "eksctl-instana-tier-cluster-cluster"
    2025-06-28 11:12:15 [!]  1 error(s) occurred and cluster hasn't been created properly, you may wish to check CloudFormation console
    2025-06-28 11:12:15 [ℹ]  to cleanup resources, run 'eksctl delete cluster --region=ap-south-1 --name=instana-tier-cluster'
    2025-06-28 11:12:15 [✖]  creating CloudFormation stack "eksctl-instana-tier-cluster-cluster": operation error CloudFormation: CreateStack, https response error StatusCode: 400, RequestID: 24185895-bf3a-4cca-bfde-f37b5e4ac144, AlreadyExistsException: Stack [eksctl-instana-tier-cluster-cluster] already exists

delete the cloudformation stack         (I did from UI console)



> ***I faced more issues****and went ahead with following solutions****which worked**
(create a new policy json file)
vim eks-full-access.json                [refer [eks-full-access.json](../stage-4-practicals/eks-custom-policies/eks-full-access.json)]


(create new policy using the json file)
aws iam create-policy \
  --policy-name <policy-name> \
  --policy-document file://<saved-filename>.json
  
ex:
aws iam create-policy \
  --policy-name CustomEKSFullAccess \
  --policy-document file://eks-full-access.json

    {
        "Policy": {
            "PolicyName": "CustomEKSFullAccess",
            "PolicyId": "ANPAUSYTFEFKMP3PUDVYT",
            "Arn": "arn:aws:iam::00518XX407XXX:policy/CustomEKSFullAccess",
            "Path": "/",
            "DefaultVersionId": "v1",
            "AttachmentCount": 0,
            "PermissionsBoundaryUsageCount": 0,
            "IsAttachable": true,
            "CreateDate": "2025-06-28T11:18:09+00:00",
            "UpdateDate": "2025-06-28T11:18:09+00:00"
        }
    }


(attach the newly policy to the user group)
aws iam attach-group-policy  --group-name <name-of-IAM-group>  --policy-arn arn:aws:iam::<user-account-id>:policy/<policy-name>
ex: aws iam attach-group-policy  --group-name VPC_EC2_EBS_EKS  --policy-arn arn:aws:iam::00518XX407XXX:policy/CustomEKSFullAccess


(then create the cluster - **refer WARNING section (down below) if encountered resource constraints of `1 worker node`**)
eksctl create cluster --name <CLUSTER_NAME> --region <REGION_NAME> --nodegroup-name <NODEGROUP_NAME>  --node-type <EC2_INSTANCE_TYPE> --nodes <NUMBER_of_NODES> --nodes-min <MIN_of_NODES>  --nodes-max <MAX_of_NODES> --managed

ex: el create cluster --name instana-tier-cluster --region ap-south-1 --nodegroup-name three-tier-app-nodes --node-type t2.medium --nodes 1 --nodes-min 1 --nodes-max 1 --managed

        2025-06-28 11:20:12 [ℹ]  eksctl version 0.210.0
        2025-06-28 11:20:12 [ℹ]  using region ap-south-1
        2025-06-28 11:20:12 [ℹ]  skipping ap-south-1c from selection because it doesn't support the following instance type(s): t2.medium
        2025-06-28 11:20:12 [ℹ]  setting availability zones to [ap-south-1a ap-south-1b]
        2025-06-28 11:20:12 [ℹ]  subnets for ap-south-1a - public:192.168.0.0/19 private:192.168.64.0/19
        2025-06-28 11:20:12 [ℹ]  subnets for ap-south-1b - public:192.168.32.0/19 private:192.168.96.0/19
        2025-06-28 11:20:12 [ℹ]  nodegroup "three-tier-app-nodes" will use "" [AmazonLinux2023/1.32]
        2025-06-28 11:20:12 [ℹ]  using Kubernetes version 1.32
        2025-06-28 11:20:12 [ℹ]  creating EKS cluster "instana-tier-cluster" in "ap-south-1" region with managed nodes
        2025-06-28 11:20:12 [ℹ]  will create 2 separate CloudFormation stacks for cluster itself and the initial managed nodegroup
        2025-06-28 11:20:12 [ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=ap-south-1 --cluster=instana-tier-cluster'
        2025-06-28 11:20:12 [ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "instana-tier-cluster" in "ap-south-1"
        2025-06-28 11:20:12 [ℹ]  CloudWatch logging will not be enabled for cluster "instana-tier-cluster" in "ap-south-1"
        2025-06-28 11:20:12 [ℹ]  you can enable it with 'eksctl utils update-cluster-logging --enable-types={SPECIFY-YOUR-LOG-TYPES-HERE (e.g. all)} --region=ap-south-1 --cluster=instana-tier-cluster'
        2025-06-28 11:20:12 [ℹ]  default addons vpc-cni, kube-proxy, coredns, metrics-server were not specified, will install them as EKS addons
        2025-06-28 11:20:12 [ℹ]  
        2 sequential tasks: { create cluster control plane "instana-tier-cluster", 
            2 sequential sub-tasks: { 
                2 sequential sub-tasks: { 
                    1 task: { create addons },
                    wait for control plane to become ready,
                },
                create managed nodegroup "three-tier-app-nodes",
            } 
        }
        2025-06-28 11:20:12 [ℹ]  building cluster stack "eksctl-instana-tier-cluster-cluster"
        2025-06-28 11:20:13 [ℹ]  deploying stack "eksctl-instana-tier-cluster-cluster"
        2025-06-28 11:20:43 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-cluster"
        2025-06-28 11:21:13 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-cluster"
        2025-06-28 11:22:13 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-cluster"
        2025-06-28 11:23:13 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-cluster"
        2025-06-28 11:24:13 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-cluster"
        2025-06-28 11:25:13 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-cluster"
        2025-06-28 11:26:13 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-cluster"
        2025-06-28 11:27:13 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-cluster"
        2025-06-28 11:28:13 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-cluster"
        2025-06-28 11:28:14 [!]  recommended policies were found for "vpc-cni" addon, but since OIDC is disabled on the cluster, eksctl cannot configure the requested permissions; the recommended way to provide IAM permissions for "vpc-cni" addon is via pod identity associations; after addon creation is completed, add all recommended policies to the config file, under `addon.PodIdentityAssociations`, and run `eksctl update addon`
        2025-06-28 11:28:14 [ℹ]  creating addon: vpc-cni
        2025-06-28 11:28:14 [ℹ]  successfully created addon: vpc-cni
        2025-06-28 11:28:15 [ℹ]  creating addon: kube-proxy
        2025-06-28 11:28:15 [ℹ]  successfully created addon: kube-proxy
        2025-06-28 11:28:15 [ℹ]  creating addon: coredns
        2025-06-28 11:28:16 [ℹ]  successfully created addon: coredns
        2025-06-28 11:28:16 [ℹ]  creating addon: metrics-server
        2025-06-28 11:28:16 [ℹ]  successfully created addon: metrics-server
        2025-06-28 11:30:16 [ℹ]  building managed nodegroup stack "eksctl-instana-tier-cluster-nodegroup-three-tier-app-nodes"
        2025-06-28 11:30:17 [ℹ]  deploying stack "eksctl-instana-tier-cluster-nodegroup-three-tier-app-nodes"
        2025-06-28 11:30:17 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-nodegroup-three-tier-app-nodes"
        2025-06-28 11:30:47 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-nodegroup-three-tier-app-nodes"
        2025-06-28 11:31:20 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-nodegroup-three-tier-app-nodes"
        2025-06-28 11:32:59 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-nodegroup-three-tier-app-nodes"
        2025-06-28 11:32:59 [ℹ]  waiting for the control plane to become ready
        2025-06-28 11:33:00 [✔]  saved kubeconfig as "/home/ubuntu/.kube/config"
        2025-06-28 11:33:00 [ℹ]  no tasks
        2025-06-28 11:33:00 [✔]  all EKS cluster resources for "instana-tier-cluster" have been created
        2025-06-28 11:33:00 [ℹ]  nodegroup "three-tier-app-nodes" has 1 node(s)
        2025-06-28 11:33:00 [ℹ]  node "ip-192-168-31-114.ap-south-1.compute.internal" is ready
        2025-06-28 11:33:00 [ℹ]  waiting for at least 1 node(s) to become ready in "three-tier-app-nodes"
        2025-06-28 11:33:00 [ℹ]  nodegroup "three-tier-app-nodes" has 1 node(s)
        2025-06-28 11:33:00 [ℹ]  node "ip-192-168-31-114.ap-south-1.compute.internal" is ready
        2025-06-28 11:33:00 [✔]  created 1 managed nodegroup(s) in cluster "instana-tier-cluster"
        2025-06-28 11:33:01 [ℹ]  kubectl command should work with "/home/ubuntu/.kube/config", try 'kubectl get nodes'
        2025-06-28 11:33:01 [✔]  EKS cluster "instana-tier-cluster" in "ap-south-1" region is ready


NOTE: `eksctl create cluster` command automatically sets the newly created EKS cluster as the current context and configures i.e, updates the ~/.kube/config file.


[NOT NEEDED: THIS CREATES ANOTHER/DUPLICATE EKS CONTEXT, ONLY DO,
i.e, update `Kubectl Context` i.e, the `kubeconfig` to effectively manage EKS cluster resources directly from terminal IF `eksctl create cluster` was not run]

aws eks update-kubeconfig --region ap-south-1 --name instana-tier-cluster

    Added new context arn:aws:eks:ap-south-1:00518XX407XXX:cluster/instana-tier-cluster to /home/ubuntu/.kube/config

> above command does the following after an EKS (Elastic Kubernetes Service) cluster is created:

- It configures the local kubeconfig file (usually at ~/.kube/config) to allow kubectl to communicate with your specified EKS cluster (named `instana-tier-cluster`) in the ap-south-1 region.

- It fetches the cluster’s access details and credentials from AWS and adds (or updates) an entry in the kubeconfig.

- After running this command, we can use kubectl commands to manage our EKS cluster’s resources directly from your terminal.

    - In short: This command connects our local kubectl tool to our new EKS cluster so we can manage it.


[TO DELETE context],
kl config delete-context <context-name>
ex: kl config delete-context arn:aws:eks:ap-south-1:00518XX407XXX:cluster/instana-tier-cluster


(see all clusters)
kl config get-contexts

    CURRENT   NAME                                                                   CLUSTER                                     AUTHINFO                                                               NAMESPACE
    *         hf-samsum-data-ingest-user@instana-tier-cluster.ap-south-1.eksctl.io   instana-tier-cluster.ap-south-1.eksctl.io   hf-samsum-data-ingest-user@instana-tier-cluster.ap-south-1.eksctl.io   
              kind-my-practicals-kind-cluster                                        kind-my-practicals-kind-cluster             kind-my-practicals-kind-cluster                                        helm-mania


(see current cluster pointed to aka current-context)
kl config current-context

    hf-samsum-data-ingest-user@instana-tier-cluster.ap-south-1.eksctl.io


(see cluster info via kubectl)
kl cluster-info

    Kubernetes control plane is running at https://191901762F528923C69A10F90478CFDC.gr7.ap-south-1.eks.amazonaws.com
    CoreDNS is running at https://191901762F528923C69A10F90478CFDC.gr7.ap-south-1.eks.amazonaws.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.


(see cluster info via aws cli)
aws eks describe-cluster --region=<REGION_NAME> --name=<CLUSTER_NAME>
ex: aws eks describe-cluster --region=ap-south-1 --name=instana-tier-cluster

    {
    "cluster": {
        "name": "instana-tier-cluster",
        "arn": "arn:aws:eks:ap-south-1:00518XX407XXX:cluster/instana-tier-cluster",
        "createdAt": "2025-06-28T11:20:39.900000+00:00",
        "version": "1.32",
        "endpoint": "https://191901762F528923C69A10F90478CFDC.gr7.ap-south-1.eks.amazonaws.com",
        "roleArn": "arn:aws:iam::00518XX407XXX:role/eksctl-instana-tier-cluster-cluster-ServiceRole-uQcHiveRyoNZ",
        "resourcesVpcConfig": {
            "subnetIds": [
                "subnet-00e4f8bddcdcc49cc",
                "subnet-01ed3293d74521426",
                "subnet-03a9baba7c0409ce7",
                "subnet-01a0b08678174be8e"
            ],
            "securityGroupIds": [
                "sg-00453ce3e2cb992c6"
            ],
            "clusterSecurityGroupId": "sg-0f163a7ce7767dffd",
            "vpcId": "vpc-051774afa6b01daa8",
            "endpointPublicAccess": true


(see nodes running in the cluster)
kl get nodes -o wide

    NAME                                            STATUS   ROLES    AGE   VERSION               INTERNAL-IP      EXTERNAL-IP    OS-IMAGE                       KERNEL-VERSION                    CONTAINER-RUNTIME
    ip-192-168-31-114.ap-south-1.compute.internal   Ready    <none>   21m   v1.32.3-eks-473151a   192.168.31.114   43.205.130.6   Amazon Linux 2023.7.20250609   6.1.140-154.222.amzn2023.x86_64   containerd://1.7.27



(see node groups running in the cluster)
el get nodegroup --cluster=<CLUSTER_NAME> --region=<REGION_NAME>
ex: el get nodegroup --cluster=instana-tier-cluster --region=ap-south-1

    CLUSTER			NODEGROUP		STATUS	CREATED			MIN SIZE	MAX SIZE	DESIRED CAPACITY	INSTANCE TYPE	IMAGE IASG NAME							TYPE
    instana-tier-cluster	three-tier-app-nodes	ACTIVE	2025-06-28T11:30:43Z	1		1		1			t2.medium	AL2023_x86_64_STANDARD	eks-three-tier-app-nodes-62cbdb26-e3bc-983b-78b7-560444d20867	managed




(see **EKS-specific cluster details**)
el get cluster --region=<REGION_NAME> --name=<CLUSTER_NAME>
ex: el get cluster --region=ap-south-1 --name=instana-tier-cluster      or, el get cluster --region=ap-south-1

    NAME			VERSION	STATUS	CREATED			VPC			SUBNETS									SECURITYGROUPS		PROVIDER
    instana-tier-cluster	1.32	ACTIVE	2025-06-28T11:20:39Z	vpc-051774afa6b01daa8	subnet-00e4f8bddcdcc49cc,subnet-01a0b08678174be8e,subnet-01ed3293d74521426,subnet-03a9baba7c0409ce7	sg-00453ce3e2cb992c6	EKS


(created a `namespace` and switched to it)
kl create ns <ns-name>
ex: kl create ns robot-shop

    namespace/robot-shop created

kl config set-context --current --namespace <ns-name>
ex: kl config set-context --current --namespace robot-shop

    Context "hf-samsum-data-ingest-user@instana-tier-cluster.ap-south-1.eksctl.io" modified.

kl get ns

    NAME              STATUS   AGE
    default           Active   61m
    robot-shop      Active   92s
    kube-node-lease   Active   61m
    kube-public       Active   61m
    kube-system       Active   61m



## configure IAM OIDC provider
export CLUSTER_NAME=<EKS-CLUSTER-NAME>
ex: export CLUSTER_NAME=instana-tier-cluster

then,
OIDC_ID=$(aws eks describe-cluster --name $CLUSTER_NAME --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)


(next check if there is an IAM OIDC provider configured already)
aws iam list-open-id-connect-providers | grep $OIDC_ID | cut -d "/" -f4             (no o/p)


(If not, run the below command)
el utils associate-iam-oidc-provider --region ap-south-1 --cluster $CLUSTER_NAME --approve

    2025-06-28 13:09:59 [ℹ]  will create IAM Open ID Connect provider for cluster "instana-tier-cluster" in "ap-south-1"
    2025-06-28 13:09:59 [✔]  created IAM Open ID Connect provider for cluster "instana-tier-cluster" in "ap-south-1"


then again,
aws iam list-open-id-connect-providers | grep $OIDC_ID | cut -d "/" -f4

        191901XXXX892XXX69XXXX0478XXC"



## configure & deploy AWS LB controller

(Download IAM policy - downloads a file `iam_policy.json` in current directory)
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json

      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
        100  8759  100  8759    0     0  22752      0 --:--:-- --:--:-- --:--:-- 22750


(Create IAM Policy)
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json

    {
    "Policy": {
        "PolicyName": "AWSLoadBalancerControllerIAMPolicy",
        "PolicyId": "ANPAUSYTFEFKPVY4UOEI4",
        "Arn": "arn:aws:iam::00518XX407XXX:policy/AWSLoadBalancerControllerIAMPolicy",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2025-06-28T13:15:00+00:00",
        "UpdateDate": "2025-06-28T13:15:00+00:00"
    }
}


(Create IAM Role)
el create iamserviceaccount \
  --cluster=$CLUSTER_NAME \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve

ex:
el create iamserviceaccount \
  --cluster=$CLUSTER_NAME \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::00518XX407XXX:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve


        2025-06-28 13:17:41 [ℹ]  1 iamserviceaccount (kube-system/aws-load-balancer-controller) was included (based on the include/exclude rules)
        2025-06-28 13:17:41 [!]  serviceaccounts that exist in Kubernetes will be excluded, use --override-existing-serviceaccounts to override
        2025-06-28 13:17:41 [ℹ]  1 task: { 
            2 sequential sub-tasks: { 
                create IAM role for serviceaccount "kube-system/aws-load-balancer-controller",
                create serviceaccount "kube-system/aws-load-balancer-controller",
            } }2025-06-28 13:17:41 [ℹ]  building iamserviceaccount stack "eksctl-instana-tier-cluster-addon-iamserviceaccount-kube-system-aws-load-balancer-controller"
        2025-06-28 13:17:41 [ℹ]  deploying stack "eksctl-instana-tier-cluster-addon-iamserviceaccount-kube-system-aws-load-balancer-controller"
        2025-06-28 13:17:41 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-addon-iamserviceaccount-kube-system-aws-load-balancer-controller"
        2025-06-28 13:18:11 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-addon-iamserviceaccount-kube-system-aws-load-balancer-controller"
        2025-06-28 13:18:11 [ℹ]  created serviceaccount "kube-system/aws-load-balancer-controller"


## deploy ALB controller

(Add helm repo & update it)
hl repo add eks https://aws.github.io/eks-charts

    "eks" has been added to your repositories

hl repo update eks

    Hang tight while we grab the latest from your chart repositories...
    ...Successfully got an update from the "eks" chart repository
    Update Complete. ⎈Happy Helming!⎈


(install the helm chart for the `load balancer controller`)
hl install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=$CLUSTER_NAME \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>


ex:
hl install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=$CLUSTER_NAME \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=ap-south-1 \
  --set vpcId=vpc-051774afa6b01daa8

        NAME: aws-load-balancer-controller
        LAST DEPLOYED: Sat Jun 28 13:23:47 2025
        NAMESPACE: kube-system
        STATUS: deployed
        REVISION: 1
        TEST SUITE: None
        NOTES:
        AWS Load Balancer controller installed!


(Verify that the `load balancer deployments` are running)
kl get deployment aws-load-balancer-controller -n kube-system

    NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
    aws-load-balancer-controller   2/2     2            2           114s



## configure **EBS CSI Plugin**

> This is needed while working with stateful applications aka in memory data store to provision a volume automatically to the persistent volume claim (PVC)

(create a `service account` - This command deploys an AWS CloudFormation stack that creates an IAM role and attaches the IAM policy to it.)

el create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster $CLUSTER_NAME \
    --role-name AmazonEKS_EBS_CSI_DriverRole \
    --role-only \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve

    2025-06-28 13:37:22 [ℹ]  1 existing iamserviceaccount(s) (kube-system/aws-load-balancer-controller) will be excluded
    2025-06-28 13:37:22 [ℹ]  1 iamserviceaccount (kube-system/ebs-csi-controller-sa) was included (based on the include/exclude rules)
    2025-06-28 13:37:22 [!]  serviceaccounts in Kubernetes will not be created or modified, since the option --role-only is used
    2025-06-28 13:37:22 [ℹ]  1 task: { create IAM role for serviceaccount "kube-system/ebs-csi-controller-sa" }
    2025-06-28 13:37:22 [ℹ]  building iamserviceaccount stack "eksctl-instana-tier-cluster-addon-iamserviceaccount-kube-system-ebs-csi-controller-sa"
    2025-06-28 13:37:23 [ℹ]  deploying stack "eksctl-instana-tier-cluster-addon-iamserviceaccount-kube-system-ebs-csi-controller-sa"
    2025-06-28 13:37:23 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-addon-iamserviceaccount-kube-system-ebs-csi-controller-sa"
    2025-06-28 13:37:53 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-addon-iamserviceaccount-kube-system-ebs-csi-controller-sa"


(create the addon)
el create addon --name aws-ebs-csi-driver --cluster $CLUSTER_NAME --service-account-role-arn arn:aws:iam::<AWS-ACCOUNT-ID>:role/AmazonEKS_EBS_CSI_DriverRole --force

ex:
el create addon --name aws-ebs-csi-driver --cluster $CLUSTER_NAME--service-account-role-arn arn:aws:iam::00518XX407XXX:role/AmazonEKS_EBS_CSI_DriverRole --force

        2025-06-28 13:42:49 [ℹ]  Kubernetes version "1.32" in use by cluster "instana-tier-cluster"
        2025-06-28 13:42:49 [ℹ]  IRSA is set for "aws-ebs-csi-driver" addon; will use this to configure IAM permissions
        2025-06-28 13:42:49 [!]  the recommended way to provide IAM permissions for "aws-ebs-csi-driver" addon is via pod identity associations; after addon creation is completed, run `eksctl utils migrate-to-pod-identity`
        2025-06-28 13:42:49 [ℹ]  using provided ServiceAccountRoleARN "arn:aws:iam::00518XX407XXX:role/AmazonEKS_EBS_CSI_DriverRole"
        2025-06-28 13:42:49 [ℹ]  creating addon: aws-ebs-csi-driver


## get the `helm chart/manifests` & deploy all resources in the current namespace,

(I used a sample repo)
git clone https://github.com/<user-name>/<repo-name>.git
mv three-tier-architecture-demo/EKS/helm/* .
rm -rf three-tier-architecture-demo


(installing the chart (i.e, everything inside `/templates` directory) from within the root directory i.e, `helm` folder)
hl install <release-name> -n <ns-name> .

ex: hl install instana-shop -n robot-shop .

    NAME: instana-shop
    LAST DEPLOYED: Sat Jun 28 14:22:19 2025
    NAMESPACE: robot-shop
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None


(watch the `pods` in same namespace)
kl get pods -n robot-shop -w

    NAME                        READY   STATUS              RESTARTS   AGE
    cart-655b74fb49-9glns       1/1     Running             0          24s
    catalogue-b4855db44-92d9h   0/1     Pending             0          24s
    dispatch-845799dc84-fqzwp   0/1     ContainerCreating   0          24s
    mongodb-69d9cf5747-pntvn    0/1     Pending             0          24s
    mysql-8c599b989-tbzl6       0/1     Pending             0          24s
    payment-6589fd67f6-v7c2d    1/1     Running             0          24s
    rabbitmq-876447689-lbwk6    1/1     Running             0          24s
    ratings-6fb5c59f44-d5nzw    0/1     Pending             0          24s
    redis-0                     1/1     Running             0          24s
    shipping-67cdd8c8c6-5njlt   0/1     Pending             0          24s
    user-b4977f556-sh78h        1/1     Running             0          24s
    web-7649bf4886-qvj2j        0/1     Pending             0          24s
    dispatch-845799dc84-fqzwp   1/1     Running             0          25s


**WARNING**: sometimes, due to resource constraints of `1 worker node` the micro-services (pods) needs more worker nodes, hence scale them

# First, update the nodegroup to allow more nodes (set max nodes to 3 for flexibility)
el scale nodegroup --cluster $CLUSTER_NAME --name three-tier-app-nodes --nodes 2 --nodes-max 3 --region ap-south-1

    2025-06-28 14:54:07 [ℹ]  scaling nodegroup "three-tier-app-nodes" in cluster instana-tier-cluster
    2025-06-28 14:54:08 [ℹ]  initiated scaling of nodegroup
    2025-06-28 14:54:08 [ℹ]  to see the status of the scaling run `eksctl get nodegroup --cluster instana-tier-cluster --region ap-south-1 --name three-tier-app-nodes`

then,
kl get pods

    NAME                        READY   STATUS    RESTARTS   AGE
    cart-655b74fb49-9glns       1/1     Running   0          23m
    catalogue-b4855db44-92d9h   1/1     Running   0          23m
    dispatch-845799dc84-fqzwp   1/1     Running   0          23m
    mongodb-69d9cf5747-pntvn    1/1     Running   0          23m
    mysql-8c599b989-tbzl6       1/1     Running   0          23m
    payment-6589fd67f6-v7c2d    1/1     Running   0          23m
    rabbitmq-876447689-lbwk6    1/1     Running   0          23m
    ratings-6fb5c59f44-d5nzw    0/1     Running   0          23m
    redis-0                     1/1     Running   0          23m
    shipping-67cdd8c8c6-5njlt   0/1     Running   0          23m
    user-b4977f556-sh78h        1/1     Running   0          23m
    web-7649bf4886-qvj2j        1/1     Running   0          23m


(see all 12 `services` deployed)
kl get svc -n robot-shop

    NAME        TYPE           CLUSTER-IP       EXTERNAL-IP                                                                 PORT(S)                       AGE
    cart        ClusterIP      10.100.176.43    <none>                                                                      8080/TCP                      25m
    catalogue   ClusterIP      10.100.56.10     <none>                                                                      8080/TCP                      25m
    dispatch    ClusterIP      None             <none>                                                                      55555/TCP                     25m
    mongodb     ClusterIP      10.100.127.243   <none>                                                                      27017/TCP                     25m
    mysql       ClusterIP      10.100.239.246   <none>                                                                      3306/TCP                      25m
    payment     ClusterIP      10.100.177.12    <none>                                                                      8080/TCP                      25m
    rabbitmq    ClusterIP      10.100.202.174   <none>                                                                      5672/TCP,15672/TCP,4369/TCP   25m
    ratings     ClusterIP      10.100.76.136    <none>                                                                      80/TCP                        25m
    redis       ClusterIP      10.100.146.165   <none>                                                                      6379/TCP                      25m
    shipping    ClusterIP      10.100.86.90     <none>                                                                      8080/TCP                      25m
    user        ClusterIP      10.100.27.68     <none>                                                                      8080/TCP                      25m
    web         LoadBalancer   10.100.94.79     k8s-robotsho-web-6561c0223a-821325d860cc56c4.elb.ap-south-1.amazonaws.com   8080:31468/TCP                25m

> we'll not expose `LoadBalancer` svc to user, rather will use an ingress


(deploy the `ingress` resource)
kl apply -f ingress.yaml

    Warning: annotation "kubernetes.io/ingress.class" is deprecated, please use 'spec.ingressClassName' instead
    ingress.networking.k8s.io/robot-shop created


(fix above error by reapplying with file: [ingress-three-tier.yaml](../stage-4-practicals/ingress-three-tier.yaml)),
kl apply -f ingress.yaml

    ingress.networking.k8s.io/robot-shop created


(get the LoadBalancer `A record`)
kl get ing

    NAME         CLASS   HOSTS   ADDRESS                                                                    PORTS   AGE
    robot-shop   alb     *       k8s-robotsho-robotsho-df75a7c7e9-1578256083.ap-south-1.elb.amazonaws.com   80      23s



(Finally, CLEAN UP clusters),
--------------
el delete cluster --region=ap-south-1 --name=instana-tier-cluster       
or, el delete cluster --name instana-tier-cluster --region ap-south-1

    2025-06-28 14:59:54 [ℹ]  deleting EKS cluster "instana-tier-cluster"
    2025-06-28 14:59:54 [ℹ]  will drain 0 unmanaged nodegroup(s) in cluster "instana-tier-cluster"
    2025-06-28 14:59:54 [ℹ]  starting parallel draining, max in-flight of 1
    2025-06-28 14:59:54 [ℹ]  deleted 0 Fargate profile(s)
    2025-06-28 14:59:54 [✔]  kubeconfig has been updated
    2025-06-28 14:59:54 [ℹ]  cleaning up AWS load balancers created by Kubernetes objects of Kind Service or Ingress
    2025-06-28 15:00:23 [ℹ]  
    3 sequential tasks: { delete nodegroup "three-tier-app-nodes", 
        2 sequential sub-tasks: { 
            2 parallel sub-tasks: { 
                2 sequential sub-tasks: { 
                    delete IAM role for serviceaccount "kube-system/ebs-csi-controller-sa",
                    delete serviceaccount "kube-system/ebs-csi-controller-sa",
                },
                2 sequential sub-tasks: { 
                    delete IAM role for serviceaccount "kube-system/aws-load-balancer-controller",
                    delete serviceaccount "kube-system/aws-load-balancer-controller",
                },
            },
            delete IAM OIDC provider,
        }, delete cluster control plane "instana-tier-cluster" [async] 
    }
    2025-06-28 15:00:23 [ℹ]  will delete stack "eksctl-instana-tier-cluster-nodegroup-three-tier-app-nodes"
    2025-06-28 15:00:23 [ℹ]  waiting for stack "eksctl-instana-tier-cluster-nodegroup-three-tier-app-nodes" to get deleted
    2025-06-28 15:00:23 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-nodegroup-three-tier-app-nodes"
    2025-06-28 15:00:53 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-nodegroup-three-tier-app-nodes"
    2025-06-28 15:01:41 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-nodegroup-three-tier-app-nodes"
    2025-06-28 15:03:16 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-nodegroup-three-tier-app-nodes"
    2025-06-28 15:04:48 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-nodegroup-three-tier-app-nodes"
    2025-06-28 15:05:55 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-nodegroup-three-tier-app-nodes"
    2025-06-28 15:06:35 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-nodegroup-three-tier-app-nodes"
    2025-06-28 15:08:09 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-nodegroup-three-tier-app-nodes"
    2025-06-28 15:08:09 [ℹ]  will delete stack "eksctl-instana-tier-cluster-addon-iamserviceaccount-kube-system-aws-load-balancer-controller"
    2025-06-28 15:08:09 [ℹ]  waiting for stack "eksctl-instana-tier-cluster-addon-iamserviceaccount-kube-system-aws-load-balancer-controller" to get deleted
    2025-06-28 15:08:09 [ℹ]  will delete stack "eksctl-instana-tier-cluster-addon-iamserviceaccount-kube-system-ebs-csi-controller-sa"
    2025-06-28 15:08:09 [ℹ]  waiting for stack "eksctl-instana-tier-cluster-addon-iamserviceaccount-kube-system-ebs-csi-controller-sa" to get deleted
    2025-06-28 15:08:09 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-addon-iamserviceaccount-kube-system-ebs-csi-controller-sa"
    2025-06-28 15:08:09 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-addon-iamserviceaccount-kube-system-aws-load-balancer-controller"
    2025-06-28 15:08:39 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-addon-iamserviceaccount-kube-system-aws-load-balancer-controller"
    2025-06-28 15:08:39 [ℹ]  waiting for CloudFormation stack "eksctl-instana-tier-cluster-addon-iamserviceaccount-kube-system-ebs-csi-controller-sa"
    2025-06-28 15:08:39 [ℹ]  serviceaccount "kube-system/ebs-csi-controller-sa" was not created by eksctl; will not be deleted
    2025-06-28 15:08:39 [ℹ]  deleted serviceaccount "kube-system/aws-load-balancer-controller"
    2025-06-28 15:08:40 [ℹ]  will delete stack "eksctl-instana-tier-cluster-cluster"
    2025-06-28 15:08:41 [✔]  all cluster resources were deleted
