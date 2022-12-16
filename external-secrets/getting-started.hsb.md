# External Secrets 

[External Secrets Operator](https://external-secrets.io/latest/) is a Kubernetes operator that integrates external secret management systems, reads information from external APIs and automatically injects the values into a Kubernetes Secret.

The External Secrets Operaror includes a number of API resources that allow yout to synchronize secrets from external APIs into your cluster and these include
1. [ExternalSecret](https://external-secrets.io/v0.7.0/introduction/overview/#externalsecret)
2. [SecretStore](https://external-secrets.io/v0.7.0/introduction/overview/#secretstore)
3. [ClusterSecretStore](https://external-secrets.io/v0.7.0/introduction/overview/#clustersecretstore) 

### Installing the External Secrets Operator 
Tanzu Application Platform packages a version of the External Secrets Operator that can be installed as follows on the `tap-install` namespace

```sh
export ESO_VERSION=0.6.1+tap.1
export TAP_NAMESPACE="tap-install"

tanzu package install external-secrets -p external-secrets.apps.tanzu.vmware.com \
  -v "${ESO_VERSION}" \
  -n "${TAP_NAMESPACE}"
```

### Connecting to a secret manager 

#### Example : Google Secret Manager 

In order to connect to a Google Secrets Manager instance you will need a Service Account with the right permissions to connect to it. The following steps detail how to Create a ClusterStore reource that uses this Service Account to connect to your Secrets Manager 

1. Create a secret that holds the service account 
```sh
ytt -f google-secrets-manager-secret.yaml  \
  --data-value gcp_secret_name=google-secrets-manager-secret  \
  --data-value secrets_namespace=external-secrets \
  --data-value-file service_account_json=<path_to_service_account_json> | kubectl apply -f- 

```
where
```yaml
❯ cat google-secrets-manager-secret.yaml
#@ load("@ytt:data", "data")
---
apiVersion: v1
kind: Secret
metadata:
  labels:
    type: gcpsm
  name: #@ data.values.gcp_secret_name
  namespace: #@ data.values.secrets_namespace
stringData:
  secret-access-credentials: #@ data.values.service_account_json
type: Opaque
```

2. Create a ClusterStoreSecret resource that references the secret above.


```sh
export GOOGLE_PROJECT_ID="project-id-name"

ytt -f google-secrets-store.yaml \
--data-value cluster_store_name=google-secrets-manager-store-secret \
--data-value secret_name=google-secrets-manager-secret-store \
--data-value google_project_id="#{GOOGLE_PROJECT_ID}" | kubectl delete -f-
```

where
```yaml
❯ cat google-secrets-store.yaml
#@ load("@ytt:data", "data")
---
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name:  #@ data.values.cluster_store_name
spec:
  provider:
    gcpsm:
      auth:
        secretRef:
          secretAccessKeySecretRef:
            key: secret-access-credentials
            name: #@ data.values.secret_name
            namespace: external-secrets
      projectID: #@ data.values.google_project_id
You will need to create a serviceaccount with the r ight access permikssions to your secrets nager a n asscoiated key t
Create a secret that holds your service account to allow you to connect to your Google Secrets Manager instance
```

To check that we have correctly authenticated to the Secrets Manager 
```sh  
  tanzu external-secrets stores list -A
  NAMESPACE  NAME                                  PROVIDER                STATUS
  Cluster    google-secrets-manager-cluster-store  Google Secrets Manager  Valid
```

### Create a Synced Secret 

The secret to be synced needs to exist on your Secret Manager or you can create one as shown below 

Consider an example where we would like to acccess our secret to access a maven repository stored in a asecret ` maven-registry-credential` in our Secret Manager. 

```sh
kubectl apply -f external-secret.yaml
```

where

Create an external secrets resource with the reference to the secret to be synced and the `Valid` ClusterStore configured above 
```yaml
---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: git-secret
  namespace: my-apps
spec:
  data:
  # SecretStoreRef defines which SecretStore to use when fetching the secret data
  - remoteRef:
      conversionStrategy: Default
      key: maven-registry-credentials
    secretKey: apitoken
  refreshInterval: 1m
  secretStoreRef:
    kind: ClusterSecretStore
    name: google-secrets-manager-cluster-store
  # the target describes the secret that shall be created
  # there can only be one target per ExternalSecret
  target:
    creationPolicy: Owner
    deletionPolicy: Retain
    name: maven-registry-credentials
    template:
      metadata:
        annotations:
          tekton.dev/git-0: https://github.com
      type: kubernetes.io/basic-auth
      data:
        password: '{{ .apitoken | toString }}'
        username: "_json_key"
      engineVersion: v2
```

### Using a Synced Secret 
#### Example: Using the synced secret to access a maven artifact.

```sh
tanzu apps wld apply $workload_name \
      --service-account supply-chain-service-account \
      --param-yaml maven='{"artifactId": "spring-petclinic", "version": "3.7.4", "groupId": "org.springframework.samples"}' \
      --param maven_repository_secret_name="maven-registry-credentials"\
      --type web \
      --app spring-petclinic -y -n my-apps
```
