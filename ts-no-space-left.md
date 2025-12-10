---

copyright:
  years: 2025
lastupdated: "2025-12-10"

keywords: troubleshooting for PostgreSQL, postgresql max_connections, postgres max connections, postgresql connection pooling, postgres connection pooling, disk space, scaling considerations

subcollection: databases-for-postgresql-gen2

content-type: troubleshoot

---

 {{site.data.keyword.attribute-definition-list}}

# How do I scale disk to get my {{site.data.keyword.databases-for-postgresql_full}} deployment back online?
{: #troubleshoot-no-space-left}
{: troubleshoot}
{: support}

[Gen 2]{: tag-purple}

{{site.data.keyword.databases-for}} Gen 2 is currently in Beta. The Beta plan is provided exclusively for evaluation and testing purposes. It is not covered by warranties, SLAs, or support, and is not intended for production use. For more information, see the  [Beta reference](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-icd-gen2-beta).
{: beta}

If you encounter a `No space left on device` error for your {{site.data.keyword.databases-for-postgresql_full}} deployment, review these solutions.
{: shortdesc}

Your deployment becomes unreachable and produces a `[Errno 28] No space left on device` error message.
{: tsSymptoms}

Review the following information to troubleshoot and resolve your `No space left on device` issues:
{: tsResolve}

* [Scale up the disk](/docs/databases-for-postgresql?topic=databases-for-postgresql-resources-scaling&interface=ui) to get your instance back online.
   Scaling is a potentially long-running operation. For more information, see [Scaling Considerations](/docs/databases-for-postgresql?topic=databases-for-postgresql-resources-scaling&interface=ui#resources-scaling-consider).

* To prevent an issue with disk space, set up [monitoring](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-monitoring).
* Autoscaling is designed to respond to the short-to-medium term trends in resource usage on your {{site.data.keyword.databases-for-postgresql_full}} deployment. When enabled, your deployment is checked at the interval you specify. For more information, see [Autoscaling](/docs/databases-for-postgresql?topic=databases-for-postgresql-autoscaling&interface=ui).
