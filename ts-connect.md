---

copyright:
  years: 2025
lastupdated: "2025-12-10"

keywords: troubleshooting for PostgreSQL, connecting postgresql deployment, postgres endpoints

subcollection: databases-for-postgresql-gen2

content-type: troubleshoot

---

{{site.data.keyword.attribute-definition-list}}

# Why can’t I connect to my PostgreSQL deployment?
{: #troubleshoot-connect}
{: troubleshoot}
{: support}

[Gen 2]{: tag-purple}

{{site.data.keyword.databases-for}} Gen 2 is currently in Beta. The Beta plan is provided exclusively for evaluation and testing purposes. It is not covered by warranties, SLAs, or support, and is not intended for production use. For more information, see the [Beta reference](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-icd-gen2-beta).
{: beta}

If you encounter errors when connecting to your {{site.data.keyword.databases-for-postgresql_full}} deployment, review these common causes and resolutions.
{: shortdesc}

You receive an error message or fail to connect to a {{site.data.keyword.databases-for-postgresql}} deployment.  If reviewing application logs, you might see errors that mention intermittent connection timeouts or unable to connect.
{: tsSymptoms}

Review the following information to troubleshoot and resolve common connectivity problems:
{: tsResolve}

* An unsecured connection is a common cause of connectivity errors.  All {{site.data.keyword.databases-for-postgresql}} connections use TLS/SSL encryption; {{site.data.keyword.databases-for-postgresql}} rejects unsecured connections.  To avoid errors, make sure you configured a secure connection.  Refer to [Getting started](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-getting-started) for an example of a secure connection.
* If you are using a private endpoint, make sure that you specify connection strings that contain the private endpoint (see [Credentials for private endpoints](/docs/cloud-databases?topic=cloud-databases-service-endpoints&interface=ui#private-endpoints-credentials)) and that you followed the steps in [Connecting through private endpoints](/docs/cloud-databases?topic=cloud-databases-service-endpoints&interface=ui#private-endpoint-connections).
* If your application log captures a short connection interruption, that behavior is expected as a normal part of operations for this managed service. You want to design your applications to retry connections when errors are caused by a temporary loss in connectivity to your deployment or to {{site.data.keyword.cloud_notm}}. However, if you experience several minutes of connection interruption check the Cloud Status for the service. For more information, see [Application-level high-availability](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-postgresql-ha-dr&interface=ui#application-level-ha).
