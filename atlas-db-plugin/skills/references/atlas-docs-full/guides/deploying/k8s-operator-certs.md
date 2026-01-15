Using SSL Certs with the Atlas Operator | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Many modern databases support SSL/TLS encryption for secure communication between clients and the database. In this document we provide some basic guidance on how to use SSL/TLS certificates with the [Atlas Operator](/integrations/kubernetes) on Kubernetes.

## Step 1: Create a Secret for the SSL/TLS Certificates[​](#step-1-create-a-secret-for-the-ssltls-certificates "Direct link to Step 1: Create a Secret for the SSL/TLS Certificates")


The first step is to create a Kubernetes Secret that contains the SSL/TLS certificates. If you are using a Kubernetes Operator that supports automatically creating certificates such as the [CockroachDB Operator](https://github.com/cockroachdb/cockroach-operator), you can use the certificates created by that Operator.

Here is an example of how to create a Secret with SSL/TLS certificates:
```codeBlockLines_AdAo
kubectl create secret generic my-secret \  --from-file=ca.crt=./path/to/ca.crt \  --from-file=tls.key=./path/to/tls.key \  --from-file=tls.crt=./path/to/tls.crt
```
This will create a Secret named `my-secret` with the SSL/TLS certificates.

## Step 2: Mount the Certificates into the Atlas Operator[​](#step-2-mount-the-certificates-into-the-atlas-operator "Direct link to Step 2: Mount the Certificates into the Atlas Operator")


The next step is to mount the SSL/TLS certificates into the Atlas Operator. To do this, by create a file named `values.yaml` with the following content:
```codeBlockLines_AdAo
extraVolumes:   - name: certs     secret:       secretName: my-secret       defaultMode: 0640extraVolumeMounts:   - name: certs     mountPath: /certs     readOnly: true
```
Now, install the operator using this `values.yaml` file:
```codeBlockLines_AdAo
helm install atlas-operator oci://ghcr.io/ariga/charts/atlas-operator -f values.yaml
```
This will install the Atlas Operator, overriding the `extraVolumes` and `extraVolumeMounts` values to mount the SSL/TLS certificates into the Operator.

## Step 3: Use the Certificates in the Database URL[​](#step-3-use-the-certificates-in-the-database-url "Direct link to Step 3: Use the Certificates in the Database URL")


The final step is to use the SSL/TLS certificates in the database [URL](/concepts/url). For example, if you are using the PostgreSQL or CockroachDB databases, you can use the following database URL:
```codeBlockLines_AdAo
postgresql://username@hostname:port/database?sslmode=verify-full&sslcert=/certs/tls.crt&sslkey=/certs/tls.key&sslrootcert=/certs/ca.crt
```
To learn more about how to securely provide the database URL to the operator, see the [docs](/guides/deploying/secrets).

*   [Step 1: Create a Secret for the SSL/TLS Certificates](#step-1-create-a-secret-for-the-ssltls-certificates)
*   [Step 2: Mount the Certificates into the Atlas Operator](#step-2-mount-the-certificates-into-the-atlas-operator)
*   [Step 3: Use the Certificates in the Database URL](#step-3-use-the-certificates-in-the-database-url)