Deploying Versioned Migrations in Kubernetes from Atlas Schema Registry | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

This guide will walk you through deploying versioned migrations in Kubernetes from Atlas Schema Registry.

Use this setup if:

*   You are using the Atlas Kubernetes Operator with the versioned migrations flow (e.g using `AtlasMigration` CRDs).
*   You have a CI/CD pipelines pushing your migration directory to the Atlas Schema Registry.

## Prerequisites[​](#prerequisites "Direct link to Prerequisites")


*   An Atlas Cloud account with a project on the Atlas Schema Registry
*   An Atlas Cloud Bot Token (see [Creating a Bot Token](/cloud/bots#creating))
*   A Kubernetes cluster
*   Helm and Kubectl installed

## Steps[​](#steps "Direct link to Steps")


1.  Create a Kubernetes Secret with your Atlas Cloud Bot Token
```codeBlockLines_AdAo
kubectl create secret generic atlas-registry-secret --from-literal=token=<your token>
```
2.  Create a Kubernetes Secret with your database credentials.
```codeBlockLines_AdAo
kubectl create secret generic db-credentials --from-literal=url="mysql://root:pass@localhost:3306/myapp"
```
Replace the `url` value with your database credentials.

3.  Install the Atlas Operator
```codeBlockLines_AdAo
helm install atlas-operator oci://ghcr.io/ariga/charts/atlas-operator
```
4.  Locate your Cloud project name in the Atlas Schema Registry

![Atlas Schema Registry](/assets/images/cloud-project-name-b1a0a2b0c0e2dcb27b1d27e36c480970.png)

Open the Project Information pane on the right and locate the project slug (e.g `project-name`) in the URL.

4.  Create an file named `migration.yaml` with the following content:

migration.yaml
```codeBlockLines_AdAo
apiVersion: db.atlasgo.io/v1alpha1kind: AtlasMigrationmetadata:  name: atlasmigrationspec:  urlFrom:    secretKeyRef:      key: url      name: db-credentials  cloud:    tokenFrom:      secretKeyRef:        key: token        name: atlas-registry-secret  dir:    remote:      name: "project-name" # Migration directory name in your atlas cloud project      tag: "latest"
```
Replace `project-name` with the name of your migration directory in the Atlas Schema Registry.

If you would like to deploy a specific version of the migrations, replace `latest` with the version tag.

5.  Apply the AtlasMigration CRD manifest
```codeBlockLines_AdAo
kubectl apply -f migration.yaml
```
6.  Check the status of the AtlasMigration CRD:
```codeBlockLines_AdAo
kubectl get atlasmigration
```
`kubectl` will output the status of the migration:
```codeBlockLines_AdAo
NAME             READY   REASONatlasmigration   True    Applied
```
7.  Observe the reported migration logs on your Cloud project in the Atlas Schema Registry:

    ![Atlas Schema Registry](/assets/images/k8s-cloud-logs-92e968d512a13969bb2f2b2a6a048374.png)

*   [Prerequisites](#prerequisites)
*   [Steps](#steps)