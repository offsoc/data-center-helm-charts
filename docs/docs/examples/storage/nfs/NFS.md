# NFS server for Bitbucket
!!!warning "Disclaimer"

    **The NFS helm chart is not officially supported. It should not be used for production deployments!**
    
    The included NFS example is provided as is and should be used as reference a only. Before you proceed we highly recommend that you understand your specific deployment needs and tailor your solution to them.

## Bitbucket Data Center and NFS

When deploying Bitbucket, you have two options how to store git data. You can deploy [Bitbucket Mesh nodes](../bitbucket/BITBUCKET_MESH.md)
or you can use shared home for your Bitbucket Data Center instance.

Due to the high performance requirements on IO operations, it is critical that you adhere to the requirements in [Bitbucket Supported platforms](https://confluence.atlassian.com/display/BitbucketServer/Supported+platforms).
 
## NFS provisioning
The NFS server can be provisioned manually or by using the supplied Helm chart. Details for both approaches can be found below.

!!!tip "Pod affinity"

    To reduce the IO latency between the NFS server and Bitbucket Pod(s) it is  highly recommend to keep them in close proximity. To achieve this, you can use [standard Kubernetes affinity rules](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity){.external}. The `affinity` stanza within `values.yaml` can be updated to take advantage of this behaviour i.e.

### Manual
For information on setting up Bitbucket Data Center's shared file server, see [Provision your shared file system](https://confluence.atlassian.com/bitbucketserver/install-bitbucket-data-center-872139817.html#InstallBitbucketDataCenter-nfs){.external}. 
This section contains the requirements and recommendations for setting up NFS for Bitbucket Data Center.

!!!tip "NFS Server sizing"

    Ensure the NFS server's size is appropriate for the needs of the Bitbucket instance. See [capacity recommendations](https://confluence.atlassian.com/bitbucketserver/recommendations-for-running-bitbucket-in-aws-776640282.html){.external} for details.

### Helm
!!!warning "Disclaimer"

    **This Helm chart is not officially supported! It should not be used for production deployments!**

#### Installation
Create a namespace for the NFS
```shell
kubectl create namespace nfs
```
Clone this repository and from the sub-directory, `data-center-helm-charts/docs/docs/examples/storage/nfs`, run the following command:
```shell
helm install nfs-server nfs-server-example --namespace nfs
```

#### Uninstall
```shell
helm uninstall nfs-server --namespace nfs
```

## Update `values.yaml`
Get the IP address of the NFS service (`CLUSTER-IP`) by running the following command
```shell
kubectl get service --namespace nfs -o jsonpath='{.items[0].spec.clusterIP}'
```
!!!info "NFS directory share"

    The NFS Helm chart creates and exposes the directory share `/srv/nfs`. This will be required when configuring `values.yaml` 
The approach below shows how a `persistentVolume` and corresponding `peristentVolumeClaim` can be dynamically created for the provisioned NFS. Using the NFS IP and directory share, (see above) update the `values.yaml` appropriately:
```yaml
volumes:
  sharedHome:
    persistentVolume:
      create: true
      nfs:
        server: "10.100.197.23" # IP address of the NFS server 
        path: "/srv/nfs" # Directory share of NFS
    persistentVolumeClaim:
      create: true
      storageClassName: ""
```
You can of course manually provision your own `persistentVolume` and corresponding claim (as opposed to the dynamic approach described above) for the NFS server. In this case update the `values.yaml` to make use of them via the `customVolume` stanza, `sharedHome.persistentVolume.create` and `sharedHome.persistentVolumeClaim.create` should also both be set to `false`.
```yaml
sharedHome:
  persistentVolume:
    create: false
  persistentVolumeClaim:
    create: false
  customVolume: 
    persistentVolumeClaim:
      claimName: "custom-nfs-server-claim"
```
---
!!!tip "Next step - Local storage"

    Having created the shared home NFS continue with provisioning the next piece of prerequisite infrastructure, [local storage](../../../userguide/PREREQUISITES.md#configure-local-home-volume).
