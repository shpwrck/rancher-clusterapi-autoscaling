# rancher-clusterapi-autoscaling

This repo outlines all of the steps required to run cluster-autoscaler with Rancher >2.6 and RKE2 provisioning.

## Requirements

- One Rancher server
- Admin Permissions

## Steps

1. Create an RKE2 cluster on a cloud provider.
   - Create one `Machine Pool` with etcd/controlplane roles enabled.
   - Create one `Machine Pool` with just worker role enabled.
2. In the Rancher management cluster create the https://kubernetes.github.io/autoscaler repository. [Included in repo](https://raw.githubusercontent.com/shpwrck/rancher-clusterapi-autoscaling/main/rancher-cluster-resources/repository.yaml)
3. Install the autoscaler application into the `fleet-default` namespace. [Example in this repo](https://raw.githubusercontent.com/shpwrck/rancher-clusterapi-autoscaling/main/rancher-cluster-resources/values.yaml)
4. Apply additional roles/rolebindings. [Example provided in repo](https://raw.githubusercontent.com/shpwrck/rancher-clusterapi-autoscaling/main/rancher-cluster-resources/additional_roles_and_bindings.yaml)
5. Clone the worker `Machine Pool` and change the following:

      Add two annotations:
      ```yaml
      ...
      metadata:
        annotations: 
          cluster.x-k8s.io/cluster-api-autoscaler-node-group-max-size: "{{ INSERT MAX SIZE DIGIT }}"
          cluster.x-k8s\.io/cluster-api-autoscaler-node-group-min-size: "{{ INSERT MIN SIZE DIGIT }}"
      ...
      ```
      Remove generated labels:
      ```yaml
      ...
      labels:
        objectset.rio.cattle.io/hash: 581f8260dac18d89c7f6a63e0e10f0dbfe0443
      ...
      ```
      Update the deployment-name label in two locations:
      ```yaml
      ...
      spec:
        selector:
          matchLabels:
            cluster.x-k8s.io/deployment-name: {{ INSERT NEW DEPLOYMENT NAME }}
        ...
        template:
          metadata:
            labels:
              cluster.x-k8s.io/deployment-name: {{ INSERT NEW DEPLOYMENT NAME }}
      ...
      ```
      Update the metadata.name:
      ```yaml
      ...
      metadata:
          name: {{ INSERT NEW DEPLOYMENT NAME }}
      ...
      ```yaml
      
6. Test with workload.

## Current Implications

- Rancher is not "fully aware" of the new node pool. This means that if you delete the cluster you may have to delete the additional machines/machinesets/machinedeployments.
- The reason for the additional node pool is that the Rancher provisioning controller overrides scaling adjustments.
