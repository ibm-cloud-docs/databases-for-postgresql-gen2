---

copyright:
  years: 2018, 2025
lastupdated: "2025-09-15"

keywords: deployment, crn, task, gui, api endpoint, postgresql dashboard

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Navigation and Overview page
{: #dashboard-overview}

## Overview page
{: #overview}

The _Overview_ page shows you information about your {{site.data.keyword.databases-for-postgresql_full}} deployment. The overview includes essential identifying information.

The _Overview_ page shows you information about your {{site.data.keyword.databases-for-postgresql_full}} deployment. The overview includes essential identifying information.

### Type
{: #type}

The type of database that is offered by the service, and the database version that your service uses.

### CRN (deployment ID)
{: #id}

The ID is a [CRN (Cloud Resource Name)](/docs/account?topic=account-crn){: external} that uniquely identifies the database instance. The CRN is used to refer to the database in the API and can be used with the CLI. The _Overview_ pane shows details of your service.

### Recent tasks
{: #recent-tasks}

Each time that you make administrative changes to your service (such as scaling, or taking a manual backup), a task starts up. The _Recent Tasks_ panel shows the task name and progress bar for any running tasks, and a list of the most recently completed tasks. Depending on how busy your deployment is, successful tasks can be shown for 24 - 48 hours. Unsuccessful tasks can show for 7 - 8 days. Tasks can also be retrieved from the [{{site.data.keyword.databases-for}} API](/apidocs/cloud-databases-api/cloud-databases-api-v5#listdeploymenttasks){: external} and [CLI plug-in](https://cloud.ibm.com/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference#deployment-tasks-list). A historical record of tasks from any time period is available through [{{site.data.keyword.atracker_full}}](/docs/databases-for-postgresql?topic=databases-for-postgresql-at_events).

### Endpoints
{: #endpoints}

The _Endpoints_ pane within the _Overview_ pane contains connection strings for your instance. Each tab contains connection information that is tailored to the type of connection or the protocol that uses it. Basic information includes _hostname_, _port_, the TLS service proprietary certificate, TLS/SSL parameters, and the default database of your instance.

Reference tables for the different connection types are available on the [Getting credentials and connection strings](/docs/databases-for-postgresql?topic=databases-for-postgresql-connection-strings) page.

Connection strings reflect whether your instance uses public endpoints, private endpoints, or both. You can configure which endpoints are available on your deployment. For more information, see the [Service endpoints integration](/docs/cloud-databases?topic=cloud-databases-service-endpoints) page.

You can manage your {{site.data.keyword.databases-for-postgresql}} service through the {{site.data.keyword.databases-for}} API. This panel provides the essential information for using the API. For more information about the {{site.data.keyword.databases-for}} API, see the [API reference](https://{DomainName}/apidocs/cloud-databases-api) page.

## Resources
{: #resources-tab}

The resources tab contains information and configuration options on the size and resource usage of your deployment. You can
- [Scale disk, memory, and CPU](/docs/databases-for-postgresql?topic=databases-for-postgresql-resources-scaling)
- [Configure autoscaling](/docs/databases-for-postgresql?topic=databases-for-postgresql-autoscaling)

## Read-only replicas
{: #read-replicas-tab}

The _Read-only replicas_ tab contains the UI for details for the read replicas for your deployment. Updates to replication leaders are asynchronously copied to their read-only replicas. For more information, see [Configuring read-only replicas](/docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas).

## Backups and restore
{: #backups-tab}

The _Backups and restore_ tab is the UI for managing your instance's backups. Available backups are listed with their timestamps. Click a backup to grab its ID or to restore it into a new deployment. For more information, see [Managing backups](/docs/cloud-databases?topic=cloud-databases-dashboard-backups).

## Observability
{: #observability-tab}

The _Observability_ tab provides access to the IBM Cloud Monitoring, logging, and event tracking integrations available for your deployment.

- [{{site.data.keyword.atracker_full}}](/docs/databases-for-postgresql?topic=databases-for-postgresql-at_events)
- [{{site.data.keyword.logs_routing_full}}](/docs/databases-for-postgresql?topic=databases-for-postgresql-logging)
- [{{site.data.keyword.monitoringfull}}](/docs/databases-for-postgresql?topic=databases-for-postgresql-monitoring)

## Settings
{: #settings-tab}

The _Settings_ tab contains the UI for many of the tunable settings for your deployment. You can:

- View encryption details. Encryption at rest is enabled for all {{site.data.keyword.databases-for-postgresql}} deployments. If you brought your own encryption key from [Key Protect](/docs/cloud-databases?topic=cloud-databases-key-protect), _Settings_ provides a link to your Key Protect instance, and the _Encryption Key_ field has the name of the key.
- [Change the admin password](/docs/databases-for-postgresql?topic=databases-for-postgresql-user-management&interface=ui#user-management-set-admin-password-ui).
- [Protect Cloud Databases with context-based restrictions](/docs/cloud-databases?topic=cloud-databases-cbr&interface=ui).
- [Implement or modify an IP allowlist](/docs/cloud-databases?topic=cloud-databases-allowlisting).

## Service credentials
{: #service-cred-tab}

You can generate a new set of credentials for cases where you want to manually [connect an application running on IBM Cloud](/docs/databases-for-postgresql?topic=databases-for-postgresql-ibmcloud-app) or an [external application](/docs/databases-for-postgresql?topic=databases-for-postgresql-external-app) to an {{site.data.keyword.cloud_notm}} service. For more information, see [Adding and viewing credentials](/docs/account?topic=account-service_credentials).


## View docs
{: #view-docs}

The _View docs_ link from the `Actions` drop list opens the main documentation page for {{site.data.keyword.databases-for-postgresql}} in a new tab.
