# Retreive and create service accounts

When you install Tanzu Application Platform, the Supply Chain Security
Tools (SCST) - Store deployment automatically includes a read-write service
account so users don't have to create it.
This service account is bound to the `metadata-store-read-write` role.

There are two types of SCST - Store service accounts:

1. Read-write service account - full access to the `POST` and `GET` API requests
2. Read-only service account - can only use `GET` API requests

This topic explains how to create service accounts.

## <a id='rw-serv-accts'></a> Create read-write service account

When you install Tanzu Application Platform, the SCST - Store deployment automatically includes a read-write service account.
This service account is already bound to the `metadata-store-read-write` role.

To create an *additional* read-write service account, run the following command.
The command creates a service account called `metadata-store-read-write-client`, depending on the Kubernetes version:

```console
kubectl apply -f - -o yaml << EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: metadata-store-read-write
  namespace: metadata-store
rules:
- resources: ["all"]
  verbs: ["get", "create", "update"]
  apiGroups: [ "metadata-store/v1" ]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: metadata-store-read-write
  namespace: metadata-store
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: metadata-store-read-write
subjects:
- kind: ServiceAccount
  name: metadata-store-read-write-client
  namespace: metadata-store
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: metadata-store-read-write-client
  namespace: metadata-store
  annotations:
    kapp.k14s.io/change-group: "metadata-store.apps.tanzu.vmware.com/service-account"
automountServiceAccountToken: false
---
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: metadata-store-read-write-client
  namespace: metadata-store
  annotations:
    kapp.k14s.io/change-rule: "upsert after upserting metadata-store.apps.tanzu.vmware.com/service-account"
    kubernetes.io/service-account.name: "metadata-store-read-write-client"
EOF
```

> **Note** For Kubernetes v1.24 and later, services account secrets are no
> longer automatically created.
> This is why the example adds a `Secret` resource in the earlier YAML.

## <a id='ro-serv-accts'></a>Create read-only service account

### With a default cluster role

During Store installation, the `metadata-store-read-only` cluster role
is created by default. This cluster role allows the bound user to have `get`
access to all resources. To bind to this cluster role, run the following command
depending on the Kubernetes version:

```console
kubectl apply -f - -o yaml << EOF
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: metadata-store-read-only
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: metadata-store-read-only
subjects:
- kind: ServiceAccount
  name: metadata-store-read-client
  namespace: metadata-store
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: metadata-store-read-client
  namespace: metadata-store
  annotations:
    kapp.k14s.io/change-group: "metadata-store.apps.tanzu.vmware.com/service-account"
automountServiceAccountToken: false
---
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: metadata-store-read-client
  namespace: metadata-store
  annotations:
    kapp.k14s.io/change-rule: "upsert after upserting metadata-store.apps.tanzu.vmware.com/service-account"
    kubernetes.io/service-account.name: "metadata-store-read-client"
EOF
```

> **Note** For Kubernetes v1.24 and later, services account secrets are no
> longer automatically created.
> This is why the example adds a `Secret` resource in the earlier YAML.

### With a custom cluster role

If using the default role is not sufficient, see [Create a service account with a custom cluster role](custom-role.hbs.md).

## Additional Resources

- [Retrieve access tokens](retrieve-access-tokens.hbs.md)
- [Create a service account with a custom cluster role](custom-role.hbs.md)
