---

copyright:
  years: 2025
lastupdated: "2025-12-16"

keywords: troubleshooting for PostgreSQL, postgresql max_connections, postgres max connections, postgresql connection pooling, postgres connection pooling

subcollection: databases-for-postgresql-gen2

content-type: troubleshoot

---

{{site.data.keyword.attribute-definition-list}}

# How do I increase the max_connections on my {{site.data.keyword.databases-for-postgresql_full}} deployment?
{: #troubleshoot-max-connect}
{: troubleshoot}
{: support}

[Gen 2]{: tag-purple}

{{site.data.keyword.databases-for}} Gen 2 is currently in Beta. The Beta plan is provided exclusively for evaluation and testing purposes. It is not covered by warranties, SLAs, or support, and is not intended for production use. For more information, see the  [Beta reference](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-icd-gen2-beta).
{: beta}

If you encounter an error when attempting to increase the `max_connections` on your {{site.data.keyword.databases-for-postgresql_full}} deployment, review these solutions.
{: shortdesc}

You receive a `superuser` error message when attempting the increase the `max_connections` on your {{site.data.keyword.databases-for-postgresql_full}} deployment.
{: tsSymptoms}

Review the following information to troubleshoot and resolve your `max_connections` issues:
{: tsResolve}

* The preferred method for this process is to use [Connection Pooling](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-managing-connections&interface=ui#connection-pooling). Connection pooling prevents you from exceeding the connection limit and ensures that connections from your applications are being handled efficiently.{: .note}
* Alternatively, see [Changing your {{site.data.keyword.databases-for-postgresql_full}} Configuration](/docs/databases-for-postgresql?topic=databases-for-postgresql-changing-configuration) and [Managing connections](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-managing-connections).
