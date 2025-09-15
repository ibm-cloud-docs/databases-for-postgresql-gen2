---

copyright:
  years: 2020, 2023
lastupdated: "2023-02-08"

keywords: troubleshooting for PostgreSQL, postgresql max_connections, postgres max connections, postgresql connection pooling, postgres connection pooling, disk space, scaling considerations

subcollection: databases-for-postgresql

content-type: troubleshoot

---
 
 {{site.data.keyword.attribute-definition-list}}

# How do I scale disk to get my {{site.data.keyword.databases-for-postgresql_full}} deployment back online?
{: #troubleshoot-no-space-left}
{: troubleshoot}
{: support}

If you encounter a `No space left on device` error for your {{site.data.keyword.databases-for-postgresql_full}} deployment, review these solutions.
{: shortdesc}

Your deployment becomes unreachable and produces a `[Errno 28] No space left on device` error message.
{: tsSymptoms}

Review the following information to troubleshoot and resolve your `No space left on device` issues:
{: tsResolve}

* [Scale up the disk](/docs/databases-for-postgresql?topic=databases-for-postgresql-resources-scaling&interface=ui) to get your instance back online. 
   Scaling is a potentially long-running operation. For more information, see [Scaling Considerations](/docs/databases-for-postgresql?topic=databases-for-postgresql-resources-scaling&interface=ui#resources-scaling-consider).

* To prevent an issue with disk space, set up [monitoring](/docs/cloud-databases?topic=cloud-databases-monitoring). 
* Autoscaling is designed to respond to the short-to-medium term trends in resource usage on your {{site.data.keyword.databases-for-postgresql_full}} deployment. When enabled, your deployment is checked at the interval you specify. For more information, see [Autoscaling](/docs/databases-for-postgresql?topic=databases-for-postgresql-autoscaling&interface=ui).
