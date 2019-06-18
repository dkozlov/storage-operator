# [Operator SDK User Guide](https://github.com/operator-framework/operator-sdk/blob/master/doc/ansible/user-guide.md)
![](https://raw.githubusercontent.com/openshift-labs/learn-katacoda/master/ansibleop/ansible-operator-overview/assets/images/ansible-op-flow.png)

# [Information Flow for Ansible Operator](https://github.com/operator-framework/operator-sdk/blob/master/doc/ansible/information-flow-ansible-operator.md)

## Prerequisites

- git
- docker version 17.03+.
- kubectl version v1.9.0+.
- ansible version v2.6.0+
- ansible-runner version v1.1.0+
- ansible-runner-http version v1.0.0+
- dep version v0.5.0+. (Optional if you aren't installing from source)
- go version v1.12+. (Optional if you aren't installing from source)
- Access to a Kubernetes v.1.9.0+ cluster.

# Project Scaffolding Layout

After creating a new operator project using
`operator-sdk new --type ansible`, the project directory has numerous generated folders and files. The following table describes a basic rundown of each file/directory.

| File/Folders   | Purpose                           |
| :---           | :--- |
| deploy | Contains a generic set of Kubernetes manifests for deploying this operator on a Kubernetes cluster. |
| roles/<kind> | Contains an Ansible Role with [Dynamic Azure Files](https://docs.microsoft.com/en-us/azure/aks/azure-files-dynamic-pv) |
| build | Contains scripts that the operator-sdk uses for build and initialization. |
| watches.yaml | Contains Group, Version, Kind, and Ansible invocation method. |

# [Testing Ansible Operators with Molecule](https://github.com/operator-framework/operator-sdk/blob/master/doc/ansible/dev/testing_guide.md)

### Requirements
To begin, you sould have:
- The latest version of the [operator-sdk](https://github.com/operator-framework/operator-sdk) installed.
- Docker installed and running
- [Molecule](https://github.com/ansible/molecule) >= v2.20 (currently that will require installation from source, `pip install git+https://github.com/ansible/molecule.git`)
- [Ansible](https://github.com/ansible/ansible) >= v2.7
- [jmespath](https://pypi.org/project/jmespath/)
- [The OpenShift Python client](https://github.com/openshift/openshift-restclient-python) >= v0.8
- An initialized Ansible Operator project, with the molecule directory present.

# Local testing

```
sudo molecule test -s test-local
```

# Deployment
```
sed -i 's|{{ REPLACE_IMAGE }}|dfkozlov/storage-operator:v0.0.1|g' deploy/operator.yaml
sed -i "s|{{ pull_policy\|default('Always') }}|Always|g" deploy/operator.yaml

sudo operator-sdk build dfkozlov/storage-operator:v0.0.1
sudo docker push dfkozlov/storage-operator:v0.0.1

kubectl create -f deploy/crds/dkozlov_v1alpha1_storage_crd.yaml
kubrctl create -f deploy/crds/dkozlov_v1alpha1_storage_azure_cr.yaml
kubectl create -f deploy/.

```

# Azure Results
```
$ kubectl get storage --all-namespaces
NAMESPACE     NAME               AGE
kube-system   example-storage    34m
kube-system   example-storage2   1m
```

```
$ kubectl get pvc --all-namespaces
NAMESPACE     NAME               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
kube-system   example-storage    Bound    pvc-d3b55842-9093-11e9-9753-7e789c216e0a   1Gi        RWX            azurefile      2m34s
kube-system   example-storage2   Bound    pvc-091e39ae-9094-11e9-9753-7e789c216e0a   1Gi        RWX            azurefile      64s
```

```
$ kubectl delete storage example-storage -n kube-system
storage.dkozlov.com "example-storage" deleted
```

```
$ kubectl get pvc --all-namespaces
NAMESPACE     NAME               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
kube-system   example-storage2   Bound    pvc-091e39ae-9094-11e9-9753-7e789c216e0a   1Gi        RWX            azurefile      108s
```
# Local	[Kubernetes IN Docker](https://github.com/kubernetes-sigs/kind) Results
```
$ kubectl get all -n kube-system -l name=storage-operator
NAME                                   READY   STATUS    RESTARTS   AGE
pod/storage-operator-97866d5b5-lqjtz   2/2     Running   0          5m29s

NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/storage-operator   ClusterIP   10.106.177.112   <none>        8383/TCP   5m27s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/storage-operator-97866d5b5   1         1         1       5m29s
```

```
$ kubectl get storage
NAME                    AGE
example-azure-storage   39s
example-gcp-storage     38s
```

```
$ kubectl get pvc
NAME                    STATUS    VOLUME                CAPACITY   ACCESS MODES   STORAGECLASS   AGE
example-azure-storage   Pending                                                   azurefile      2m35s
example-gcp-storage     Bound     example-gcp-storage   2T         RWX                           2m32s
```
