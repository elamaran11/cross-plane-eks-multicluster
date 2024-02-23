# Cross Plane Multi EKS Cluster Management

This repo is to demonstrates the process of :
- Setting up CrossPlane with AWS Provider for EKS.
- Connecting and installing EKS Addons to remote EKS cluster from a management cluster.
- Setting up CrossPlane K8s provider to acccess and deploy k8s resource to remote EKS cluster from a management cluster.
- Flux deployment is a bonus.


Following are the details steps :


1. Install [UXP](https://github.com/upbound/universal-crossplane) on your management cluster.
2. First create an IRSA role either manually or using `eksctl`. Check [this](https://docs.upbound.io/providers/provider-aws/authentication/#create-an-iam-oidc-provider-1) doc for IRSA OIDC provide creation.
3. Run the `1_controller_config.yaml` and `2_provider_eks.yaml` in the `aws-provider-manifests` folder to setup the Cross Plane AWS provider.
4. Run the `3_eks_access_child_cluster.yaml` which will download the `kubeconfig` of the target cluster to a K8s secret in `upbound-system` namespace.
5. Run the `4_eks_addon.yaml` to create an EKS Addon-on on target cluster using CrossPlane. This approach is the only approach incase of ARM64 nodes because UXP is not supported in ARM64 nodes.
6. Next step is adding the IRSA role created in step 1 to `aws-auth` configmap target EKS cluster. Incase of cross account, your need to chain a roleARN in providerconfig, cause that role to be used to access resources in target cluster. And you will need to add that role to the target cluster aws-auth configmap, see https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html. Your source cluster IRSA role should also have permissions to assume that role you specify in roleARN. (This is the key part to get this working!!!).
7. Run the `5_k8s_provider.yaml`, `6_k8s_providerconfig.yaml` and `7_k8s_namespace.yaml` to setup K8s provider and create a K8s resource in target cluster.
8. Finally checkout the Flux `GitRepository` and `Kustomization` in `flux-target-manifests` folder as bonus to deploy K8s resource via flux/GitOps on target cluster. 