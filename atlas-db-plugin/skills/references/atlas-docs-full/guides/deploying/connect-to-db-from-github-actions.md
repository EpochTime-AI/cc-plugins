Connecting to your database from GitHub Actions | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

When using Atlas with GitHub Actions, there are scenarios where you may want Atlas to directly interact with your database. To enable this, your database must be accessible from the GitHub Actions runners. Depending on your environment and security preferences, there are several methods to facilitate this connection.

### 1\. Publicly Accessible Database[​](#1-publicly-accessible-database "Direct link to 1. Publicly Accessible Database")


If your database is publicly accessible, you can use its public IP address or hostname to establish a connection. This approach requires minimal setup and allows Atlas to interact with the database over the internet.

This method used to be considered a security misstep due to the inherent risks of exposing databases to the public internet. However, with the proliferation of cloud-native databases and security best practices, this approach is now considered acceptable in many scenarios and is the default offering for many cloud database services such as [Neon Postgres](https://neon.tech) and [Clickhouse Cloud](https://clickhouse.com).

If you choose this method, ensure proper access controls, such as strong passwords or IAM authenticatoin and SSL/TLS encryption, are in place.

### 2\. Allow-listed IP Addresses[​](#2-allow-listed-ip-addresses "Direct link to 2. Allow-listed IP Addresses")


For databases that are not publicly accessible, you can allow-list the IP addresses of GitHub Actions runners. GitHub provides a `meta` API endpoint that lists the CIDR ranges used by its runners. You can fetch this data with the following command:
```codeBlockLines_AdAo
curl https://api.github.com/meta | jq .actions
```
Once you have the IP ranges, update your database's firewall or access control settings to permit connections only from these ranges. This method ensures that only GitHub Actions runners can access your database, adding an extra layer of security compared to a publicly accessible setup.

### 3\. Self-hosted Runners[​](#3-self-hosted-runners "Direct link to 3. Self-hosted Runners")


For workflows requiring higher security or control, you can deploy self-hosted runners within your own Virtual Private Cloud (VPC). These runners operate within your private network, enabling a secure connection to your database without exposing it to the public internet.

Self-hosted runners are ideal for environments with stringent security policies or compliance requirements. For guidance on setting up and managing self-hosted runners, see the [GitHub documentation](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners).

### Conclusion[​](#conclusion "Direct link to Conclusion")


Allowing Atlas to connect directly to your database in GitHub Actions workflows enhances flexibility but requires careful consideration of security. Publicly accessible databases offer simplicity but increase exposure, while allow-listed IPs and self-hosted runners provide greater control and security. Choose the method that best balances your workflow needs with your infrastructure’s security requirements.

*   [1\. Publicly Accessible Database](#1-publicly-accessible-database)
*   [2\. Allow-listed IP Addresses](#2-allow-listed-ip-addresses)
*   [3\. Self-hosted Runners](#3-self-hosted-runners)
*   [Conclusion](#conclusion)