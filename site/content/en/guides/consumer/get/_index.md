---
title: "Get a remote package"
linkTitle: "Get"
weight: 1
type: docs
description: >
    Fetch a package from a remote git repository and apply its contents to
    a cluster
---

*Any git directory containing configuration files may be used by kpt
as a package.*

## Topics

[kpt pkg get], [Kptfile]

## `kpt pkg get` explained

Following is a short explanation of the command that will be demonstrated
in this guide.

- Get copies the `staging/cockroachdb` subdirectory from the
  [kubernetes examples] git repo
  - Since a [Kptfile] is not included with the package, a new one is created
    for the local package
- (Optional) commit the package and push to the team git repo
- Apply the package to a cluster
  - May be pushed from the local package copy (manual) or from the team repo
    (GitOps automation)

{{< svg src="images/get-command" >}}

## Steps

1. [Fetch a remote package](#fetch-a-remote-package)
2. [View the Kptfile](#view-the-kptfile)
3. [View the package contents](#view-the-package-contents)
4. [Apply the package to a cluster](#apply-the-package-to-a-cluster)
5. [View the applied package](#view-the-applied-package)

## Fetch a remote package

Packages are **fetched from remote git repository subdirectories** using
[kpt pkg get].  This guide will use the [kubernetes examples] repository
as a public package catalogue.

##### Command

```sh
kpt pkg get https://github.com/kubernetes/examples/staging/cockroachdb cockroachdb
```

##### Output

```
fetching package staging/cockroachdb from https://github.com/kubernetes/examples to cockroachdb
```

The contents of the `staging/cockroachdb` subdirectory in the
`https://github.com/kubernetes/examples` were copied to the local folder
`cockroachdb`.

{{% pageinfo color="info" %}}
- Any git subdirectory containing configuration (e.g. `deploy.yaml`) may be
  fetched and used as a package
- The local directory name that the package is copied to does NOT need to
  match the upstream directory name it is copied from
- including `.git` as part of the repo name is optional but good practice to
  ensure the repo + subdirectory are parsed correctly by the tool.
{{% /pageinfo %}}

## View the Kptfile

##### Command

The upstream commit and branch / tag reference are stored in the package's
[Kptfile].  These are used by `kpt pkg update`.

```sh
cat cockroachdb/Kptfile
```

Print the `Kptfile` written by `kpt pkg get` to see the upstream package data.

##### Output

```
apiVersion: kpt.dev/v1alpha1
kind: Kptfile
metadata:
    name: cockroachdb
upstream:
    type: git
    git:
        commit: 629c9459a9f25e468cce8af28350a03e62c5f67d
        repo: https://github.com/kubernetes/examples
        directory: staging/cockroachdb
        ref: master
```

## View the package contents

The primary package artifacts are Kubernetes [resource configuration]
(e.g. YAML files), however packages may also include supporting
artifacts such as documentation.

##### Command

```sh
tree cockroachdb/
```

##### Output

```sh
cockroachdb/
├── Kptfile
├── OWNERS
├── README.md
├── cockroachdb-statefulset.yaml
├── demo.sh
└── minikube.sh

0 directories, 6 files
```

The cockroachdb package fetched from [kubernetes examples] contains a
`cockroachdb-statefulset.yaml` file with the resource configuration, as well
as other files included in the directory.  `

The package contains 2 resource configuration files -- `deploy.yaml` and
`service.yaml`.  These are the same files that would typically be applied with
`kubectl apply`.

{{% pageinfo color="info" %}}
kpt pkg get` created a `Kptfile` since one did not exist
(for storing package state).  If the upstream package already defines a
Kptfile, then `kpt pkg get` will update the Kptfile copied from
upstream rather than replacing it.
{{% /pageinfo %}}

##### Command

```sh
head cockroachdb/cockroachdb-statefulset.yaml
```

##### Output

```sh
apiVersion: v1
kind: Service
metadata:
  # This service is meant to be used by clients of the database. It exposes a
  # ClusterIP that will automatically load balance connections to the different
  # database pods.
  name: cockroachdb-public
  labels:
    app: cockroachdb
spec:
  ports:
```

This package contains `cockroachdb/cockroachdb-statefulset.yaml` as plain old
resource configuration (e.g. YAML) -- nothing special here.  Other kpt packages
may have configuration which uses comments to attach metadata to
specific resources or fields.

## Apply the package to a cluster

Use `kubectl apply` to deploy the local package to a remote cluster.

##### Command

```sh
kubectl apply -R -f cockroachdb
```

##### Output

```sh
service/cockroachdb-public created
service/cockroachdb created
poddisruptionbudget.policy/cockroachdb-budget unchanged
statefulset.apps/cockroachdb created
```

{{% pageinfo color="info" %}}
This guide used `kubectl apply` to demonstrate how kpt packages work out of the
box with tools that have been around since the beginning of Kubernetes.

Kpt also provides the next-generation set of Kubernetes apply commands under
the [kpt live] command.
{{% /pageinfo %}}


## View the applied package

Once applied to the cluster, the remote resources can be displayed using
the common tools such as `kubectl get`.

##### Command

```sh
kubectl get all
```

##### Output

```sh
NAME                READY   STATUS    RESTARTS   AGE
pod/cockroachdb-0   1/1     Running   0          54s
pod/cockroachdb-1   1/1     Running   0          41s
pod/cockroachdb-2   1/1     Running   0          27s

NAME                         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)              AGE
service/cockroachdb          ClusterIP   None         <none>        26257/TCP,8080/TCP   55s
service/cockroachdb-public   ClusterIP   10.48.2.5    <none>        26257/TCP,8080/TCP   55s
service/kubernetes           ClusterIP   10.48.0.1    <none>        443/TCP              26m

NAME                           READY   AGE
statefulset.apps/cockroachdb   3/3     54s
```

[kubernetes examples]: https://github.com/kubernetes/examples
[resource configuration]: https://kubernetes.io/docs/concepts/configuration/overview/#general-configuration-tips
[kpt pkg get]: ../../..//reference/pkg/get
[Kptfile]: ../../../api-reference/kptfile
[kpt live]: ../../../reference/live
