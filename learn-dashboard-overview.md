---

copyright:
  years: 2025
lastupdated: "2025-11-03"

keywords: deployment, crn, task, gui, api endpoint, postgresql dashboard, Gen 2

subcollection: databases-for-postgresql-gen2

---

{{site.data.keyword.attribute-definition-list}}

# Overview page
{: #dashboard-overview}

The _Overview_ page shows you information about your {{site.data.keyword.databases-for-postgresql_full}} deployment. The overview includes essential identifying information.

## Deployment details
{: #console-overview-deployment-details}

- **Type:** The type of database that is offered by the service, and the database version that your service uses.
- **Version:** Displays the current version of your deployed database.
- **Encryption details:** Shows the encryption configuration for your backup and disk. These settings are fixed and cannot be changed after deployment.
- **Location:** Specifies the region where your database is deployed.
- **Platform:** Displays the deployment generation. For more information, see [Overview of Gen 1 and Gen 2](docs/cloud-databases-gen2?topic=cloud-databases-gen2-overview-gen1-gen2&interface=ui).
- **Hosting model:** Displays the hosting model used for your database (for example, [Isolated compute on Gen 2](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-isolated-compute&interface=ui)).
- **Resource group:** Shows the resource group selected during deployment.
- **CRN (deployment ID):** The ID is a [CRN (Cloud Resource Name)](/docs/account?topic=account-crn) that uniquely identifies the database deployment. The CRN is used to refer to the database in the API and can be used with the CLI. The _Overview_ panel shows details of your service.

## Resources
{: #console-overview-resources}

The resources tab contains information and configuration options on the size and resource usage of your deployment. You can [scale disk, memory, and CPU](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-resources-scaling&interface=ui).

## Most recent tasks
{: #console-overview-recent-tasks}

Every time you make administrative changes to your service (such as scaling, or taking a manual backup), a task starts up. The _Most recent tasks_ panel shows only the **latest task**, including its name and progress bar. After completion, the most recent task remains visible for a short period of time.

To view more than the latest task, you can configure an [{{site.data.keyword.logs_full}}](/docs/cloud-logs?topic=cloud-logs-getting-started){: external} instance to capture and retain logs from your deployment.  

A historical record of tasks from any time period is available through [{{site.data.keyword.atracker_full}}](/docs/databases-for-mongodb-gen2?topic=databases-for-mongodb-gen2-at_events). Tasks can also be retrieved programmatically from the [{{site.data.keyword.databases-for}} API](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-api#api-deployment) and [CLI plug-in](https://cloud.ibm.com/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference#deployment-tasks-list)(WHO WORKS ON NEW CLI DOC?).


## Observability
{: #console-overview-observability}

The _Observability_ tab provides access to the {{site.data.keyword.monitoringlong}}, logging, and event tracking integrations available for your deployment.

- [{{site.data.keyword.logs_full}}](/docs/databases-for-mongodb-gen2?topic=databases-for-mongodb-gen2-logging)
- [{{site.data.keyword.monitoringfull}}](/docs/databases-for-mongodb-gen2?topic=databases-for-mongodb-gen2-monitoring)


## Service endpoints
{: #console-overview-endpoints}

The _Service endpoints_ pane within the _Overview_ pane contains connection strings for your deployment.

For Gen 2 deployments, endpoints are **private only**, ensuring that your database is accessible exclusively within your {{site.data.keyword.cloud}} private network.

The information displayed includes:

- Endpoint string for connection.
- The _hostname_ and _port_ values for direct connections to the database members.
- The default database for your service.
- SSL mode parameters. Gen 2 uses certificates from *Let’s Encrypt* instead of proprietary TLS certificates. The recommended setting is **verify-full for secure connections**.

Gen 2 deployments do not include a pre-provisioned database admin password. To begin working with your instance, you must [create a service credential](/docs/databases-for-mongodb-gen2?topic=databases-for-mongodb-gen2-console-overview#create-service-cred), which provides the authentication details and connection strings required for your applications.  
For more information on reference tables for the different connection types, see [Getting credentials and connection strings](docs/databases-for-mongodb-gen2?topic=databases-for-mongodb-gen2-connection-strings&interface=ui).
{: note}


## Connect page
{: #console-connect}

In Gen 2 deployments, only private endpoints are used to enhance security. This page provides multiple connection options, accessible via tabs:

- **How-to tab**: Learn how to set up a secure connection to your database from scratch. This guide walks you through connecting via an {{site.data.keyword.cloud}} Virtual Private Cloud (VPC) using a Virtual Server Instance (VSI) with a `psql` install, and accessing the database through an {{site.data.keyword.cloud}} VPN (Virtual Private Network gateway). You can use any VSI in your account and need to create or use a service credential. You use the service endpoint details from the *Overview* page.

- **Manage connections tab**: Find links to documentation on how to manage or delete existing connections from your {{site.data.keyword.cloud}} database instance.

   For detailed ways of connecting, go to the connect docs or open the console's (UI) [Connect page](need link). 

## Backups and restore
{: #backups-tab}

The _Backups and restore_ tab is the UI for managing your instance's backups. Available backups are listed with their timestamps. Click a backup to grab its ID or to restore it into a new deployment. For more information, see [Managing backups](/docs/cloud-databases?topic=cloud-databases-dashboard-backups).

## Settings
{: #console-overview-settings}

The _Settings_ tab contains the UI for many of the tunable settings for your deployment.

- **View encryption details:** Encryption at rest is enabled for all {{site.data.keyword.databases-for-postgresql}} deployments. are automatically encrypted at rest. Disks and backups are encrypted, and the encryption keys are managed automatically by {{site.data.keyword.cloud}}. If you brought your own encryption key from [{{site.data.keyword.keymanagementserviceshort}}](/docs/databases-for-mongodb-gen2?topic=databases-for-mongodb-gen2-key-protect&interface=ui), the _Settings_ provides a link to your {{site.data.keyword.keymanagementserviceshort}} instance and displays the key name in the _Encryption key_ field.
- **Service endpoints:** View and manage network access to your deployment. By default, only private endpoints are enabled to restrict access. For more information, see [Private endpoints on Gen 2](/docs/databases-for-mongodb?topic=databases-for-mongodb-service-endpoints&interface=ui-----needs-correct-link/include links need fixing).
- **Context-based restrictions (CBR):** You can create context-based restriction rules to control access to your deployment. For more information, see [Context-based restrictions](/docs/databases-for-mongodb?topic=databases-for-mongodb-cbr&interface=ui---neeeds-correct-link/include links need fixing).
- **Disconnect all active connections:** You can disconnect all client connections to your database. This action immediately ends all active sessions and can cause service interruptions. Use this option with caution when you need to revoke access or apply configuration changes.

## Service credentials
{: #console-overview-service-cred}

Service credentials provide the authentication details that you need to connect your applications to a {{site.data.keyword.databases-for-postgresql}} deployment.  

For Gen 2 deployments, you must create a service credential before you can begin working with your instance. Unlike Gen 1, there is no pre-provisioned database admin password. On first use (or if a credential has been deleted), the _Overview_ page prompts you to create a credential in order to connect.

### Creating a service credential
{: #create-service-cred}

1. Go to the **Service credentials** tab.  
2. Click **Create credential**.  
3. (Optional) Enable **Control by Secrets Manager** to integrate the credential with your {{site.data.keyword.secrets-manager_full}} instance.  
4. Provide a name for the credential.  
5. (Optional) Expand **Advanced options** to upload a JSON file or add inline configuration parameters.  
6. Click **Create**.

The new credential appears in the list and can be copied into your applications. Credentials are generated as **one-time view only**, so be sure to store them securely after creation.

### Using service credentials
{: #using-service-cred}

In Gen 2 deployments, **service credentials are required as the first step to connect**. Unlike Gen 1, an admin password is not provided by default. If no credential exists, you are prompted to create one on the _Overview_ page before you can establish any database connection.
{: note}

- A service credential contains the username, password, and connection strings needed to connect to your database.  
- The credentials can be used directly in your application, or integrated with {{site.data.keyword.secrets-manager_full}} for centralized key management.  
- If you delete a credential, you must create a new one to continue connecting.  

For more information, see [Managing service credentials](/docs/databases-for-mongodb-gen2?topic=databases-for-mongodb-gen2-console-overview&interface=ui#console-overview-service-cred).

## View docs
{: #view-docs}

The **View docs** link from the *Actions* drop list opens the main documentation page for {{site.data.keyword.databases-for-postgresql}} in a new tab.
