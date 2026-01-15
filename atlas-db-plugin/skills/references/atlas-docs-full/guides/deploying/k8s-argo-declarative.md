GitOps for Database Schema Management with Argo CD and Atlas Kubernetes Operator (Declarative Flow) | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

[GitOps](https://opengitops.dev/) is a software development and deployment methodology that uses Git as the central repository for both code and infrastructure configurations, enabling automated and auditable deployments.

[Argo CD](https://argoproj.github.io/cd/) is a Kubernetes-native continuous delivery tool that implements GitOps principles. It uses a declarative approach to deploy applications to Kubernetes, ensuring that the desired state of the application is always maintained.

[Kubernetes Operators](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) are software extensions to Kubernetes that enable the automation and management of complex, application-specific operational tasks and domain-specific knowledge within a Kubernetes cluster.

In this guide, we will demonstrate how to use the [Atlas Kubernetes Operator](/integrations/kubernetes) and Argo CD to achieve a GitOps-based deployment workflow for your database schema.

info

This guide demonstrates the **declarative workflow** using Atlas. If you prefer a versioned migration approach, check out the [versioned workflow guide](/guides/deploying/k8s-argo).

## Pre-requisites[​](#pre-requisites "Direct link to Pre-requisites")


*   An Atlas Cloud schema setup connected to your CI. If you don't have one yet, follow the [declarative migrations guide](/declarative/setup-cicd).
*   An Atlas Cloud Bot Token (see [Creating a Bot Token](/cloud/bots#creating))
*   A running Kubernetes cluster - For learning purposes, you can use [Minikube](https://minikube.sigs.k8s.io/docs/start/), which is a tool that runs a single-node Kubernetes cluster inside a VM on your laptop.
*   [kubectl](https://kubernetes.io/docs/tasks/tools/) - a command-line tool for interacting with Kubernetes clusters.
*   [Helm](https://helm.sh/docs/intro/install/) - a package manager for Kubernetes.

## High-level architecture[​](#high-level-architecture "Direct link to High-level architecture")


Before we dive into the details of the deployment flow, let's take a look at the high-level architecture of our application.

![Application Architecture](/assets/images/app-diagram-b7221bb20c05e2f91e1f92597f75e1c6.png)

On a high level, our application consists of the following components:

1.  A backend application - in our example we will use a plain NGINX server as a placeholder for a real backend application.
2.  A database - in our example we will use a MySQL pod for the database. In a more realistic scenario, you might want to use a managed database service like AWS RDS or GCP Cloud SQL.
3.  An `AtlasSchema` Custom Resource that defines the database schema and is managed by the Atlas Operator.

In our application architecture, we have a database that is connected to our application and managed using an Atlas CR (Custom Resource). The database plays a crucial role in storing and retrieving data for the application, while the Atlas CR provides seamless integration and management of the database schema within our Kubernetes environment.

## How should you run schema changes in an Argo CD deployment?[​](#how-should-you-run-schema-changes-in-an-argo-cd-deployment "Direct link to How should you run schema changes in an Argo CD deployment?")


Integrating GitOps practices with a database in our application stack poses a unique challenge.

Argo CD provides a declarative approach to GitOps, allowing us to define an Argo CD application and seamlessly handle the synchronization process. By pushing changes to the database schema or application code to the Git repository, Argo CD automatically syncs those changes to the Kubernetes cluster.

However, as we discussed in the [introduction](/guides/deploying/intro#running-migrations-as-part-of-deployment-pipelines), ensuring the proper order of deployments is critical. In our scenario, the database deployment must succeed before rolling out the application to ensure its functionality. If the database deployment encounters an issue, it is essential to address it before proceeding with the application deployment.

Argo CD provides [Sync Waves](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-waves/) and Sync Hooks as features that help to control the order in which manifests are applied within an application. Users may add an annotation to each resource to specify in which "wave" it should be applied. Argo CD will then apply the resources in the order of the waves.

By using annotations with specific order numbers, you can determine the sequence of manifest applications. Lower numbers indicate the earlier application and negative numbers are also allowed.

To ensure that database resources are created and applied before our application, we will utilize Argo CD Sync Waves. The diagram shows our application deployment order:

![Application Architecture](/assets/images/deployment-flow-1082d9274b762b0e7ac904e3b1cf7cbe.png)

To achieve the above order, we'll annotate each resource with a sync wave annotation order number:
```codeBlockLines_AdAo
metadata:  annotations:    argocd.argoproj.io/sync-wave: "<order-number>"
```
For more information refer to the [official documentation](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-waves/).

With the theoretical background out of the way, let’s take a look at a practical example of how to deploy an application with Argo CD and the Atlas Operator.

## Installation[​](#installation "Direct link to Installation")


### 1\. Install Argo CD[​](#1-install-argo-cd "Direct link to 1. Install Argo CD")


To install Argo CD run the following commands:
```codeBlockLines_AdAo
kubectl create namespace argocdkubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
Wait until all the pods in the `argocd` namespace are running:
```codeBlockLines_AdAo
kubectl wait --for=condition=ready pod --all -n argocd
```
`kubectl` will print something like this:
```codeBlockLines_AdAo
pod/argocd-application-controller-0 condition metpod/argocd-applicationset-controller-69dbc8585c-6qbwr condition metpod/argocd-dex-server-59f89468dc-xl7rg condition metpod/argocd-notifications-controller-55565589db-gnjdh condition metpod/argocd-redis-74cb89f466-gzk4f condition metpod/argocd-repo-server-68444f6479-mn5gl condition metpod/argocd-server-579f659dd5-5djb5 condition met
```
For more information or if you run into some error refer to the [Argo CD Documentation](https://argo-cd.readthedocs.io/en/stable/getting_started/).

### 2\. Install the Atlas Operator[​](#2-install-the-atlas-operator "Direct link to 2. Install the Atlas Operator")

```codeBlockLines_AdAo
helm install atlas-operator oci://ghcr.io/ariga/charts/atlas-operator
```
Helm will print something like this:
```codeBlockLines_AdAo
Pulled: ghcr.io/ariga/charts/atlas-operator:0.1.9Digest: sha256:4dfed310f0197827b330d2961794e7fc221aa1da1d1b95736dde65c090e6c714NAME: atlas-operatorLAST DEPLOYED: Tue Jun 27 16:58:30 2023NAMESPACE: defaultSTATUS: deployedREVISION: 1TEST SUITE: None
```
Wait until the `atlas-operator` pod is running:
```codeBlockLines_AdAo
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=atlas-operator -n default
```
`kubectl` will print something like this:
```codeBlockLines_AdAo
pod/atlas-operator-866dfbc56d-qkkkn condition met
```
For more information on the installation process, refer to the [Atlas Operator Documentation](/integrations/kubernetes).

## Define the application manifests[​](#define-the-application-manifests "Direct link to Define the application manifests")


### 1\. Set up a Git repo[​](#1-set-up-a-git-repo "Direct link to 1. Set up a Git repo")


Argo CD works by tracking changes to a Git repository and applying them to the cluster, so let's set up a Git repository to serve as the central storage for all your application configuration.

### 2\. Create Atlas Cloud token secret[​](#2-create-atlas-cloud-token-secret "Direct link to 2. Create Atlas Cloud token secret")


Create a Kubernetes secret to store your Atlas Cloud API token.
```codeBlockLines_AdAo
kubectl create secret generic atlas-credentials --from-literal=token=<your atlas cloud token>
```

### 3\. Define the database resources[​](#3-define-the-database-resources "Direct link to 3. Define the database resources")


Recall that in our first sync-wave, we want to deploy the database resources to our cluster. For the purposes of this example we're deploying a simple MySQL pod to our cluster, but in a realistic scenario, you will probably want to use a managed database service such as AWS RDS, GCP Cloud SQL, or one of the available database operators for Kubernetes.

In your repository, create a new directory named `manifests` and under it create a new file named `db.yaml`:

manifests/db.yaml
```codeBlockLines_AdAo
apiVersion: v1kind: Servicemetadata:   annotations:      argocd.argoproj.io/sync-wave: "0"   name: mysqlspec:   ports:      - port: 3306   selector:      app: mysql   clusterIP: None---apiVersion: apps/v1kind: Deploymentmetadata:   annotations:      argocd.argoproj.io/sync-wave: "0"   name: mysqlspec:   selector:      matchLabels:         app: mysql   template:      metadata:         labels:            app: mysql      spec:         containers:            - image: mysql:8              name: mysql              env:                 - name: MYSQL_ROOT_PASSWORD                   value: pass                 - name: MYSQL_DATABASE                   value: example              readinessProbe:                 tcpSocket:                    port: 3306                 initialDelaySeconds: 10                 periodSeconds: 10              livenessProbe:                 tcpSocket:                    port: 3306                 initialDelaySeconds: 15                 periodSeconds: 15              ports:                 - containerPort: 3306                   name: mysql
```

### 4\. Create the AtlasSchema Custom Resource[​](#4-create-the-atlasschema-custom-resource "Direct link to 4. Create the AtlasSchema Custom Resource")


Create the AtlasSchema custom resource to define the desired schema for your database, refer to the [Atlas Operator documentation](/integrations/kubernetes/declarative), and determine the specifications, such as the desired database schema, configuration options, and additional parameters.

The schema is defined using a schema URL that points to a declarative schema stored in the Atlas Schema Registry.

manifests/schema.yaml
```codeBlockLines_AdAo
apiVersion: db.atlasgo.io/v1alpha1kind: AtlasSchemametadata:  annotations:    argocd.argoproj.io/sync-wave: "1"  name: myappspec:  url: mysql://root:pass@mysql:3306/example  cloud:    tokenFrom:      secretKeyRef:        key: token        name: atlas-credentials  schema:    url: atlas://declarative-flow?tag=7e0d7c798a15c1d3788dc515b87925f7bb00d217 # replace with your schema URL
```

### 5\. Create your backend application deployment[​](#5-create-your-backend-application-deployment "Direct link to 5. Create your backend application deployment")


For the purpose of this guide, we will deploy a simple NGINX server to act as a placeholder for a real backend server. Notice that we annotate the backend deployment with a sync wave order number of 2. This informs Argo CD to deploy the backend application after the Atlas CR is deployed and confirmed to be in healthy.

manifests/app.yaml
```codeBlockLines_AdAo
apiVersion: apps/v1kind: Deploymentmetadata:  annotations:    argocd.argoproj.io/sync-wave: "2"  name: nginxspec:  selector:    matchLabels:      app: nginx  replicas: 2  template:    metadata:      labels:        app: nginx    spec:      containers:      - name: nginx        image: nginx        ports:        - containerPort: 80
```

### 6\. Create a custom health check for Atlas objects[​](#6-create-a-custom-health-check-for-atlas-objects "Direct link to 6. Create a custom health check for Atlas objects")


To decide whether a SyncWave is complete and the next SyncWave can be started, Argo CD performs a health check on the resources in the current SyncWave. If the health check fails, Argo CD will not proceed with the next SyncWave.

Argo CD has built-in health assessment for standard Kubernetes types, such as `Deployment` and `ReplicaSet`, but it does not have a built-in health check for custom resources such as `AtlasSchema`.

To bridge this gap, Argo CD supports custom health checks written in [Lua](https://lua.org), allowing us to define our custom health assessment logic for the Atlas custom resource.

To define the custom logic for the Atlas object in Argo CD, we can add the custom health check configuration to the _**argocd-cm**_ ConfigMap. This ConfigMap is a global configuration for Argo CD that should be placed in the same namespace as the rest of the Argo CD resources. Below is a custom health check for the Atlas object:

argocd-cm.yaml
```codeBlockLines_AdAo
apiVersion: v1kind: ConfigMapmetadata:  name: argocd-cm  namespace: argocd  labels:    app.kubernetes.io/name: argocd-cm    app.kubernetes.io/part-of: argocddata:  resource.customizations: |    db.atlasgo.io/AtlasSchema:      health.lua: |        hs = {}        if obj.status ~= nil then          if obj.status.conditions ~= nil then            for i, condition in ipairs(obj.status.conditions) do              if condition.type == "Ready" and condition.status == "False" then                hs.status = "Degraded"                hs.message = condition.message                return hs              end              if condition.type == "Ready" and condition.status == "True" then                hs.status = "Healthy"                hs.message = condition.message                return hs              end            end          end        end        hs.status = "Progressing"        hs.message = "Waiting for reconciliation"        return hs
```

### 7\. Create the Argo CD Application manifest[​](#7-create-the-argo-cd-application-manifest "Direct link to 7. Create the Argo CD Application manifest")


Finally, create an Argo CD `application.yaml` file which defines our Argo application:

application.yaml
```codeBlockLines_AdAo
apiVersion: argoproj.io/v1alpha1kind: Applicationmetadata:  name: atlas-argocd-demo  namespace: argocdspec:  source:    path: manifests    repoURL: 'https://github.com/<YOUR_ORG>/<YOUR_REPO>' # <-- replace with your repo URL    targetRevision: master # <-- replace with your mainline  destination:    namespace: default    server: 'https://kubernetes.default.svc'  project: default  syncPolicy:    automated:      prune: true      selfHeal: true    retry:      limit: 5      backoff:        duration: 5s        maxDuration: 3m0s        factor: 2    syncOptions:      - CreateNamespace=true
```

## Deploying[​](#deploying "Direct link to Deploying")


Make sure all of these files are pushed to your Git repository. If you want to follow along you can use the [example repository](https://github.com/rotemtam/atlas-argocd-demo) for this guide.

### 1\. Apply the custom health check[​](#1-apply-the-custom-health-check "Direct link to 1. Apply the custom health check")


Before deploying our application, we need to apply the custom health check configuration to the Argo CD ConfigMap.
```codeBlockLines_AdAo
kubectl apply -f argocd-cm.yaml -n argocd
```

### 2\. Deploy our application[​](#2-deploy-our-application "Direct link to 2. Deploy our application")


With the custom health check in place, we can now deploy our application.
```codeBlockLines_AdAo
kubectl apply -f application.yaml
```
Once you create an Argo CD application, Argo automatically pulls the configuration files from your Git repository and applies them to your Kubernetes cluster. As a result, the corresponding resources are created based on the manifests. This streamlined process ensures that the desired state of your application is synchronized with the actual state in the cluster.

To verify the application is successfully deployed and the resources are healthy:
```codeBlockLines_AdAo
kubectl get -n argocd applications.argoproj.io atlas-argocd-demo -o=jsonpath='{range .status.resources[*]}{"\n"}{.kind}: {"\t"} {.name} {"\t"} ({.status}) {"\t"} ({.health}){end}'
```
`kubectl` will print something like this:
```codeBlockLines_AdAo
Service:       mysql   (Synced)    ({"status":"Healthy"})Deployment:    mysql   (Synced)    ({"status":"Healthy"})Deployment:    nginx   (Synced)    ({"status":"Healthy"})AtlasSchema:   myapp   (Synced)    ({"message":"The schema has been applied successfully. Apply response: {\"Changes\":{}}","status":"Healthy"})%
```

### 3\. Access the Argo CD UI[​](#3-access-the-argo-cd-ui "Direct link to 3. Access the Argo CD UI")


To view your application in the Argo CD UI, you first need to access it.

First, expose the Argo CD server using port-forwarding:
```codeBlockLines_AdAo
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Next, retrieve the initial admin password:
```codeBlockLines_AdAo
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
Now you can access the Argo CD UI by navigating to [https://localhost:8080](https://localhost:8080) in your browser. Login with:

*   Username: `admin`
*   Password: (the password you retrieved from the previous command)

note

You may need to accept the self-signed certificate warning in your browser.

Finally, we can view the health, dependency, status of all the resources on the Argo CD UI:

![Argo CD UI](/assets/images/argo-flow-1eb3df1c0f1819b0364139706a623875.png)

## Connect CI to CD[​](#connect-ci-to-cd "Direct link to Connect CI to CD")


To complete the GitOps workflow, you need to connect your CI pipeline to your Git repository. This way, whenever you push changes to your database schema, your CI pipeline will automatically lint, test, and commit the updated schema manifest to your Git repository.

After [`atlas schema push`](/integrations/github-actions#arigaatlas-actionschemapush) runs in your CI pipeline, add a step that commits and pushes the updated tag in your `schema.yaml` manifest back to your Git repository. This ensures that Argo CD will detect the change and deploy the new migration version.

Here's an example GitHub Actions workflow that demonstrates this pattern:

.github/workflows/ci-atlas.yaml
```codeBlockLines_AdAo
- uses: ariga/atlas-action/schema/push@v1  if: github.ref == 'refs/heads/main'  with:    url: "file://schema.sql"    schema-name: 'my-app'    - name: Update schema.url tag in schema.yaml  run: |    SHA="${GITHUB_SHA}"    sed -i \    "s|\(schema\.url: .*tag=\)[^[:space:]]*|\1${SHA}|" \     manifests/schema.yaml- name: Commit and push changes  run: |    git config user.name "github-actions[bot]"    git config user.email "github-actions[bot]@users.noreply.github.com"    git add manifests/schema.yaml    git commit -m "chore: update Atlas schema tag to ${GITHUB_SHA}" || exit 0    git push
```
This workflow ensures that:

1.  Schema changes are pushed to the Atlas Schema Registry.
2.  The `schema.yaml` manifest is updated with the new tag corresponding to the latest commit.
3.  The changes are committed and pushed back to the Git repository, triggering Argo CD to deploy the updated schema.

## Videos[​](#videos "Direct link to Videos")


### Database CI/CD Pipelines with ArgoCD and Atlas


### Octopus Deploy Webinar


## Conclusion[​](#conclusion "Direct link to Conclusion")


In this guide, we demonstrated how to use Argo CD to deploy an application that uses the Atlas Operator to manage the lifecycle of the database schema. We also showed how to use Argo CD's custom health check to ensure that the schema changes were successfully applied before deploying the backend application.

Using the techniques described in this guide, you can now integrate schema management into your CI/CD pipeline and ensure that your database schema is always in sync with your application code.

*   [Pre-requisites](#pre-requisites)
*   [High-level architecture](#high-level-architecture)
*   [How should you run schema changes in an Argo CD deployment?](#how-should-you-run-schema-changes-in-an-argo-cd-deployment)
*   [Installation](#installation)
    *   [1\. Install Argo CD](#1-install-argo-cd)
    *   [2\. Install the Atlas Operator](#2-install-the-atlas-operator)
*   [Define the application manifests](#define-the-application-manifests)
    *   [1\. Set up a Git repo](#1-set-up-a-git-repo)
    *   [2\. Create Atlas Cloud token secret](#2-create-atlas-cloud-token-secret)
    *   [3\. Define the database resources](#3-define-the-database-resources)
    *   [4\. Create the AtlasSchema Custom Resource](#4-create-the-atlasschema-custom-resource)
    *   [5\. Create your backend application deployment](#5-create-your-backend-application-deployment)
    *   [6\. Create a custom health check for Atlas objects](#6-create-a-custom-health-check-for-atlas-objects)
    *   [7\. Create the Argo CD Application manifest](#7-create-the-argo-cd-application-manifest)
*   [Deploying](#deploying)
    *   [1\. Apply the custom health check](#1-apply-the-custom-health-check)
    *   [2\. Deploy our application](#2-deploy-our-application)
    *   [3\. Access the Argo CD UI](#3-access-the-argo-cd-ui)
*   [Connect CI to CD](#connect-ci-to-cd)
*   [Videos](#videos)
*   [Conclusion](#conclusion)