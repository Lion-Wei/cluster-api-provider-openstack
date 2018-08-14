# Cluster-api enhancement proposal

We were started to take part in cluster-api work server weeks ago, here is the background and other work proposal.

##### Background

1. CCE realized cluster management capability, and recently publishedcluster management interface.
2. After a long period of division, the community finally come out astandard of cluster management. In the future, community will use cluster-apito represent/manage k8s cluster status and lifecycle.
3. Currently, cluster-api repo is not very vigorous, not enough to bethe actual standard. We hope give more contribution to the repo, promotecluster-api to be the actual standard, and make it benefit for Huawei Cloud.

##### Goals

1. Contribute the implementation of Huawei cloud provider, increase theactivity of the cluster-api, and then opensource k8s users can use original k8sin HuaweiCloud ECS. 
2. Enrich the implementation of cluster-api in each cloud vendor,promote extract public fields from providerConfig to cluster/machine spec. Infuture, users can use cluster-api manage k8s clusters on each cloud.

3. Make sure the extract-public-fields work is beneficial toHuaweiCloud.(so make it easier for CCE cluster manager support cluster-APIinterface)


4.       More specific contribution points and promotion plan need to bediscussed in open source group.

And we already made some achievement, which include build a new repo `cluster-api-provider-openstack`, and realized basic cluster management functions in openstack cloud provider.

Based on the original goals, here several enhancement we can do in next step.

- Machine in-place update
- Sync machine status
- Support deploy HA cluster
- Support cluster auto-scaling



### Machine in-place update

##### Background

A "Machine" is the declarative spec for a Node, as represented in Kubernetes core. If the Machine's spec is updated, a provider-specific controller is responsible for updating the Node in-place or replacing the host with a new one matching the updated spec.

For now, openstack machine-controller will replace the machine on Machine's spec update, which is not very effective. And actually there are many different scenarios in Machine update, should be treat differently.

##### Proposal

Support in-place machine update for most machine update occasions. 

| Occasion                                 | Operation                       |
| ---------------------------------------- | ------------------------------- |
| Update kubelet version                   | Kubeadm update                  |
| Update providerConfig(e.g. name/flavor/network) | Invoke openstack update api     |
| Update machine belonged cluster          | Kubeadm reset/kubeadm join      |
| Update machine roles(master<--->node)    | kubeadm reset/kubeadm init/join |



### Sync Machine Status

##### Background

We not implied update any machine status info yet.

##### Proposal

Get machine status informations from provider and update periodically. Those informations should include:

- Machine Address
- Last update time
- Version(kubelet version/control plan version)
- Machine status



### Deploy HA cluster

##### Background

Cluster-api aims to build a set of Kubernetes cluster management APIs to enable common cluster lifecycle operations (install, upgrade, repair, delete) across disparate environments. And Highly Available Clusters is definitely a common demand.

Cluster-api is using `kubeadm init` to configure a cluster master, `kubeadm` already supported "create high available cluster" in 1.11(not GA yet), so make cluster-api support deploy HA cluster is both reasonable and feasible.

##### Proposal

Cluster-api side

1. Add a field to `Cluster.Spec` to explicit cluster is HA cluster or not.
2. Some how mark cluster's master-machines and worker-machines.

Machine-controller side

1. Identify HA cluster.
2. Use `kubeadm` configure HA cluster master.



### Cluster auto-scaling

##### Background

We already have a project [kubernetes/autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler) can automatically adjusts the size of the Kubernetes cluster when:

- there are pods that failed to run in the cluster due to insufficient resources.
- some nodes in the cluster are so underutilized, for an extended period of time, that they can be deleted and their pods will be easily placed on some other, existing nodes.

Besides, openstack cloud provider can easier scale the number of machines by using "elastic cloud server"(ECS in huawei cloud). 

#####  Open Questions

- How to register scaled machines to apiserver, we don't have this mechanism yet.
- Do we need a new resource `MachineGroup`, so we don't need concern how many machines we have exactly, we only need maintain the MachineGroup template and the auto-scale strategy.
- How do the new nodes join in the cluster.



### Other long term proposal

- Extract public field, which should include:
  - Machine name
  - Image/system
  - root volume/data volume
  - Network





