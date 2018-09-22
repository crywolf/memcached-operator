# Memcached Operator

## Overview 

This Memcached operator is an operator for Kubernetes built with [operator-sdk](https://github.com/operator-framework/operator-sdk) and can be managed by the [Operator Lifecycle Manager](https://github.com/operator-framework/operator-lifecycle-manager).
 
* **Operator SDK**: Allows developers to build an Operator based on their expertise without requiring knowledge of Kubernetes API complexities.
* **Operator Lifecycle Manager**: Helps to install, update, and generally manage the lifecycle of all of the Operators (and their associated services) running across clusters.


## Deploying the Operator

> **Requirements**: Operator Lifecycle Manager must be installed in the cluster

Deploying an Operator is as simple as applying the Operator’s manifest to the desired namespace in the cluster.

```sh
$ kubectl apply -f deploy/csv-manifests/memcachedoperator.0.0.1.csv.yaml
$ kubectl get ClusterServiceVersions
```

After applying this manifest, nothing has happened yet, because the cluster does not meet the requirements specified in our manifest. Create the CustomResourceDefinition and RBAC rules for the Memcached type managed by the Operator:

```sh
$ kubectl apply -f deploy/rbac.yaml
$ kubectl apply -f deploy/operator.yaml
```


### Creating an application instance

The Memcached Operator is now running in the `memcached` namespace. Users interact with Operators via instances of CustomResources; in this case, the resource has the Kind `Memcached`. Native Kubernetes RBAC also applies to CustomResources, providing administrators control over who can interact with each Operator.

Creating instances of Memcached in this namespace will now trigger the Memcached Operator to instantiate pods running the memcached server that are managed by the Operator. The more CustomResources you create, the more unique instances of Memcached will be managed by the Memcached Operator running in this namespace.

```sh
$ kubectl apply -f deploy/memcached-app-manifests/
```


## Building an Operator after changes in source files using the Operator SDK

> **Requirements**: Operator SDK must be installed on the development machine

```sh
$ operator-sdk build quay.io/example/memcached-operator:v0.0.2
$ docker push quay.io/example/memcached-operator:v0.0.2
```


### Update an application

Manually applying an update to the Operator is as simple as creating a new Operator manifest with a `replaces` field that references the old Operator manifest. The Operator Lifecycle Manager will ensure that all resources being managed by the old Operator have their ownership moved to the new Operator without fear of any programs stopping execution. It is up to the Operators themselves to execute any data migrations required to upgrade resources to run under a new version of the Operator.

```sh
$ kubectl apply -f memcachedoperator.0.0.2.csv.yaml
$ kubectl apply -f deploy/operator.yaml
```
Kubernetes deployment manifests are generated in `deploy/operator.yaml`. The deployment image is set to the container image specified above.
