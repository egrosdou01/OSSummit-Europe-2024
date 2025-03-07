# OSSummit Europe 2024

## Introduction
The repository is dedicated to the [OSSummit Europe](https://events.linuxfoundation.org/open-source-summit-europe/) conference presentation of [Sveltos](https://github.com/projectsveltos).

## How to

### Assumptions
- We assume a Kubernetes management cluster is already in place with **Sveltos** installed. Sveltos installation details can be found [here](https://projectsveltos.github.io/sveltos/getting_started/install/install/).
- We assume the Kubernetes managed clusters are in place with **no CNI** in place
- We assume [Flux](https://fluxcd.io/flux/installation/) is installed in the management cluster and synced to a repository that holds the Kyverno YAML manifest files

### Prerequisites
- Install `kubectl` and `sveltosctl`
- Obtain the `kubeconfig` of the clusters

**Note**: Download the `sveltosctl` [here](https://github.com/projectsveltos/sveltosctl/releases).

### Management Cluster - Create Required Namespaces
```bash
$ kubectl apply -f resources/namespaces/ns_test.yaml,resources/namespaces/ns_prod.yaml
```

### Management Cluster - Register Clusters with Sveltos

```bash
$ ./sveltosctl register cluster \
--namespace=test --cluster=test01 \
--kubeconfig=/path/to/kubeconfig --labels=env=test # Register the managed cluster with Sveltos. We use the sveltosctl for simplicity

$ ./sveltosctl register cluster \
--namespace=prod --cluster=prod01 \
--kubeconfig=/path/to/kubeconfig --labels=env=prod # Register the managed cluster with Sveltos. We use the sveltosctl for simplicity
```

### Validate Registration

```bash
$ kubectl get sveltosclusters -A --show-labels # Once Sveltos is installed in the management cluster, we can get the `sveltosclusters` resources
```

### Management Cluster - Deploy Cilium

```bash
$ kubectl apply -f resources/test_cluster/clusterprofile_cilium.yaml,resources/prod_cluster/clusterprofile_cilium.yaml
```

### Validate Deployment

**Management Cluster**

```bash
$ ./sveltosctl show addons --namespace=test # Check Sveltos Deployments `test` namespace
$ ./sveltosctl show addons --namespace=prod # Check Sveltos Deployments `prod` namespace
```

**Managed Clusters**

```bash
$ export KUBECONFIG=/path/to/test-kubeconfig
$ kubectl get pods -n kube-system -w | grep -E 'cilium|hubble'
$ kubectl get ds/cilium -n kube-system -o jsonpath='{.spec.template.spec.containers[0].image}'
```

```bash
$ export KUBECONFIG=/path/to/prod-kubeconfig
$ kubectl get pods -n kube-system -w | grep -E 'cilium|hubble'
$ kubectl get ds/cilium -n kube-system -o jsonpath='{.spec.template.spec.containers[0].image}'
```

### Management Cluster - Deploy Kyverno and Cluster Policies

**Note**: For a full deployment of Sveltos and Flux, have a look [here](https://medium.com/@eleni.grosdouli/5-step-approach-projectsveltos-integration-with-flux-5e68fb584a3c).

```bash
$ kubectl apply -f resources/test_cluster/clusterprofile_kyverno.yaml
$ kubectl apply -f resources/prod_cluster/clusterprofile_kyverno.yaml
```

### Validate Deployment

```bash
$ ./sveltosctl show addons --namespace=test # Check the Sveltos Deployments `test` namespace
$ ./sveltosctl show addons --namespace=prod # Check the Sveltos Deployments `prod` namespace
```

**Note**: Feel free to update the Helm chart versions mentioned in the manifest files.

### Managed Cluster - Validate Kyverno Policies

```bash
$ export KUBECONFIG=/path/to/test-kubeconfig
$ kubectl get ClusterPolicy -A
```

```bash
$ export KUBECONFIG=/path/to/prod-kubeconfig
$ kubectl get ClusterPolicy -A
```
