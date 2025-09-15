---

copyright:
  years: 2019, 2025
lastupdated: "2025-09-15"

keywords: databases-for-postgresql release notes

subcollection: databases-for-postgresql

content-type: release-note

---

{{site.data.keyword.attribute-definition-list}}

# Release notes
{: #postgresql-relnotes}

Use these release notes to learn about the latest updates to {{site.data.keyword.databases-for-postgresql_full}} that are grouped by _date_ or _build number_.
{: shortdesc}

## 29 April 2025
{: #databases-for-postgresql-29Apr2025}
{: release-note}

Seamless Vector Search comes to {{site.data.keyword.databases-for-postgresql}} with pgVector
:  {{site.data.keyword.databases-for-postgresql_full}} users can now enhance their PostgreSQL instances with the pgVector extension, enabling native support for vector embeddings. This tight integration simplifies the development of AI-powered applications, accelerates the process, and reduces architectural complexity by eliminating the need for separate vector databases. For more information, see [Managing PostgreSQL extensions](/docs/databases-for-postgresql?topic=databases-for-postgresql-extensions).

PgVector is only offered for PostgreSQL version 15 and above.
{: important}

## 28 April 2025
{: #databases-for-postgresql-28Apr2025}
{: release-note}

{{site.data.keyword.databases-for-postgresql}} version 13 End of life on October 22, 2025
:  This note is an update to the PostgreSQL v13 end-of-life approach, replacing our September 27, 2024 publication.

:  Action is required before October 22, 2025, for your PostgreSQL v13 deployments. After October 22, 2025, all {{site.data.keyword.cloud_notm}} Databases for PostgreSQL instances on version 13 that are still active will be upgraded in-place to the next major version, version 14. We recommend completing the upgrades before the end-of-life date. For more information, see [Upgrading to a new major version](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading). 
By proactively upgrading, you can control the timing and minimize any potential downtime. If you have any questions or concerns, contact [{{site.data.keyword.databases-for}} support](https://cloud.ibm.com/unifiedsupport/supportcenter){: external}.


## 10 March 2025
{: #databases-for-postgresql-10mar2025}
{: release-note}

{{site.data.keyword.databases-for-postgresql}} {{site.data.keyword.satelliteshort}} plan is deprecated
:   {{site.data.keyword.databases-for-postgresql}} {{site.data.keyword.satelliteshort}} plan is now deprecated. As of March 10 2025, all documentation relating to {{site.data.keyword.databases-for-postgresql}} {{site.data.keyword.satelliteshort}} plan has been removed, as well as the ability to select {{site.data.keyword.databases-for-postgresql}} {{site.data.keyword.satelliteshort}} plan in the Cloud console.

## 15 November 2024
{: #databases-for-postgresql-15nov2024}
{: release-note}

{{site.data.keyword.databases-for}} logs and events are now available on {{site.data.keyword.logs_routing_full}}
: {{site.data.keyword.databases-for}} has onboarded {{site.data.keyword.logs_routing_full}}, a scalable logging service that persists logs and provides users with capabilities for querying, tailing, and visualizing logs. Customers are expected to use {{site.data.keyword.logs_routing_full}} to review their database logs and events starting **November 15, 2024**. For more information, see [Set up logging and monitoring](/docs/databases-for-postgresql?topic=databases-for-postgresql-getting-started-cdb-logging-monitoring) and [About IBM Cloud Logs](/docs/cloud-logs?topic=cloud-logs-about-cl).

## 22 October 2024
{: #databases-for-postgresql-22oct2024}
{: release-note}

{{site.data.keyword.databases-for-postgresql}} end-of-life procedure change
:  Action is required for your {{site.data.keyword.databases-for-postgresql}} version 12 deployments before January 22, 2025. After this date, all {{site.data.keyword.databases-for-postgresql}} instances on version 12 that are still active will be upgraded in-place to the next major version, version 13. 
We recommend that you complete the upgrades before the end-of-life date. For more information, see [Upgrading to a new major version](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading){: external}. By proactively upgrading, you can control the timing and minimize any potential downtime. If you have any questions or concerns, contact [{{site.data.keyword.databases-for}} support](https://cloud.ibm.com/unifiedsupport/supportcenter){: external}.

## 27 September 2024
{: #databases-for-postgresql-27sept2024}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} version 13 End of life on October 22, 2025
:  Action required before October 22, 2025, for your PostgreSQL v13 deployments. After October 22, 2025, all {{site.data.keyword.cloud_notm}} Databases for PostgreSQL instances on version 13 that are still active will be disabled. Users are expected to be on the latest versions of PostgreSQL or get the preferred version of {{site.data.keyword.databases-for-postgresql_full}}.
Upgrade your database instances by following the procedure at [Upgrading to a new Major Version](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading){: external}. If you have any questions or concerns, contact [{{site.data.keyword.databases-for}} support](https://cloud.ibm.com/unifiedsupport/supportcenter){: external}.

## 16 September 2024
{: #databases-for-postgresql-16sept2024}
{: release-note}

Private endpoints as new default
:  To ensure best possible security for your databases, private endpoints are now the default in the {{site.data.keyword.cloud}} console. CLI and Terraform now require the endpoint type to be provided as part of creating an instance.

## 1 May 2024
{: #databases-for-postgresql-01may2024}
{: release-note}

New hosting models
:  You can choose between two hosting models: Isolated Compute and Shared Compute. Isolated Compute is a secure single-tenant offering for complex, highly performant enterprise workloads. Shared Compute is a flexible multi-tenant offering for dynamic, fine-tuned, and decoupled capacity selections. For more information, see [Hosting models](/docs/cloud-databases?topic=cloud-databases-hosting-models&interface=ui){: external}.

## 18 January 2024
{: #databases-for-postgresql-18jan2023}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} version 12 End of life on January 22, 2025
:  Action required before January 22, 2025, for your PostgreSQL v12 deployments. After January 22, 2025, all {{site.data.keyword.cloud_notm}} Databases for PostgreSQL instances on version 12 that are still active will be disabled. Users are expected to be on the latest versions of PostgreSQL or get the preferred version of {{site.data.keyword.databases-for-postgresql_full}}.
Upgrade your database instances by following the procedure at [Upgrading to a new Major Version](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading){: external}. If you have any questions or concerns, contact [{{site.data.keyword.databases-for}} support](https://cloud.ibm.com/unifiedsupport/supportcenter){: external}.

## 27 November 2023
{: #databases-for-postgresql-27nov2023}
{: release-note}

Monitoring Integration documentation updated
:  Monitoring Integration documentation now lists metrics for all {{site.data.keyword.databases-for}} services. For more information, see [Monitoring Integration](/docs/cloud-databases?topic=cloud-databases-monitoring){: external}.

## 17 Nov 2023
{: #databases-for-postgresql-25aug2023}
{: release-note}

Deploy pgadmin using Code Engine and connect to your {{site.data.keyword.databases-for-postgresql}} instance published
:  With this tutorial, deploy pgadmin using Code Engine and connect to your {{site.data.keyword.databases-for-postgresql}} instance. pgadmin is a web interface that allows you to view and modify the data in your PostgreSQL database. Code Engine is a a fully managed, serverless platform that allows you to run workloads without worrying about deploying infrastructure.
For more information, see [Deploy pgadmin using Code Engine and connect to your {{site.data.keyword.databases-for-postgresql}} instance](/docs/databases-for-postgresql?topic=databases-for-postgresql-pgadmin-code-engine-icd-postgresql){: external}.

## 25 Aug 2023
{: #databases-for-postgresql-25aug2023}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} version 11 End of life on February 29, 2024
:  Action required before February 29, 2024, for your PostgreSQL 11 deployments. After February 29, 2024, all {{site.data.keyword.databases-for-postgresql_full}} instances on version 11 that are still active will be disabled. Users are expected to be on the latest versions of PostgreSQL or get the preferred version of {{site.data.keyword.databases-for-postgresql_full}}, with all the great features added to version 15. Upgrade your database instances by following the procedure at [Upgrading to a new Major Version
](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading){: external}.
For more information, see the original [announcement](https://www.ibm.com/blog/announcement/ibm-cloud-databases-for-postgresql-11-end-of-life-on-february-29-2024/){: external}.
If you have any questions or concerns, contact [{{site.data.keyword.databases-for}} support](https://cloud.ibm.com/unifiedsupport/supportcenter){: external}.

## 07 July 2023
{: #databases-for-postgresql-13july2023}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} v15 Preferred
:  For more information, see [Upgrading to a new Major Version](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading){: external}.

## 25 May 2023
{: #databases-for-postgresql-22may2023}
{: release-note}

Setting up disk alerts for disk utilization tutorial
:  In this tutorial, you use the {{site.data.keyword.cloud_notm}} API and the [{{site.data.keyword.cloud_notm}} CLI](https://cloud.ibm.com/docs/cli?topic=cli-getting-started){: external} to set up an alert that emails you whenever the disk utilization of your database exceeds 90%. This specific example creates an alert on a {{site.data.keyword.databases-for-elasticsearch}} deployment, but it is applicable to all the databases in the IBM {{site.data.keyword.databases-for}} catalog. For more information, see [Setting up disk alerts for disk utilization](/docs/cloud-databases?topic=cloud-databases-disk-util-alert-tutorial).

## 28 October 2022
{: #databases-for-postgresql-28oct2022}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} v10: End-of-Life Extension
:  To ensure additional time for you to upgrade, {{site.data.keyword.databases-for}} is extending the EOL support for PostgreSQL v10 until February 22, 2023.
Despite the extension, we strongly recommend our customers take prompt action and upgrade instances immediately. If no action is taken, databases will be disabled on February 22, 2023.

## 19 October 2022
{: #databases-for-postgresql-19oct2022}
{: release-note}

Deploying and Connecting a Cloud Databases Instance Tutorial
:  This tutorial guides you through the process of deploying a {{site.data.keyword.databases-for}} instance and connecting it to a web front end by creating a webpage that allows visitors to input a word and its definition. These values are then stored in a database running on {{site.data.keyword.databases-for}}. You install the database infrastructure by using Terraform and your web application uses the popular Express framework. The application can then be run locally, or by using Docker. For more information, see [Deploying and Connecting a {{site.data.keyword.databases-for}} Instance](/docs/cloud-databases?topic=cloud-databases-create-instance-tutorial).

## 11 October 2022
{: #databases-for-postgresql-11oct2022}
{: release-note}

Protecting {{site.data.keyword.databases-for-postgresql_full}} resources with context-based restrictions
:  Context-based restrictions (CBR) give account owners and administrators the ability to define and enforce access restrictions for {{site.data.keyword.cloud}} resources based on the context of access requests. Access to {{site.data.keyword.databases-for}} resources can be controlled with CBR and identity and access management (IAM) policies. For more information, see [Protecting Cloud Databases resources with context-based restrictions](/docs/cloud-databases?topic=cloud-databases-cbr&interface=ui).

## 27 June 2022
{: #databases-for-postgresql-27june2022}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} 10: End-of-Life in November 2022
:  On 3 November 2022, all {{site.data.keyword.databases-for-postgresql_full}} instances on version 10 that are still active will be disabled. For more information, see [{{site.data.keyword.databases-for-postgresql_full}} 10: End-of-Life in November 2022](https://www.ibm.com/cloud/blog/announcements/ibm-cloud-databases-for-postgresql-10-end-of-life-in-november-2022).

## 31 May 2022
{: #databases-for-postgresql-31may2022}
{: release-note}

Provision an {{site.data.keyword.databases-for-postgresql_full}} instance with Terraform tutorial published
:  In this tutorial, you learn how to use Terraform to provision an {{site.data.keyword.databases-for-postgresql_full}} instance. For more information, see [Provision an {{site.data.keyword.databases-for-postgresql_full}} instance with Terraform](/docs/cloud-databases?topic=cloud-databases-tutorial-provision-postgres-tf&interface=ui).

## 29 March 2022
{: #databases-for-postgresql-29mar2022}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} supports synchronous replication
:  {{site.data.keyword.databases-for-postgresql_full}} now supports synchronous replication. See documentation [here](/docs/databases-for-postgresql?topic=databases-for-postgresql-postgresql-ha-dr#ha-synchronous-replication).

## 28 March 2022
{: #databases-for-postgresql-28mar2022}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} Requirements for upgrading to PostgreSQL (v 13, 14) from PostgreSQL (v10, 11, 12)
:  See information regarding updating to versions 13 and 14 [here](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading#upgrading-reqs).

## 25 March 2022
{: #databases-for-postgresql-25mar2022}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} Security Goals Updated
:  Available security and compliance goals updated. See updated goals [here](/docs/cloud-databases?topic=cloud-databases-manage-security-compliance).

## 30 June 2021
{: #databases-for-postgresql-30jun2021}
{: release-note}

General Availability of {{site.data.keyword.databases-for-postgresql_full}} support for {{site.data.keyword.cloud_notm}} Databases enabled by {{site.data.keyword.cloud_notm}} Satellite.
:  A distributed cloud provides consistent security and services across environments, centralized workload visibility, reduced latency, easier compliance, and higher application development velocity. See blog post announcement [here](https://www.ibm.com/cloud/blog/announcements/deploy-managed-cloud-native-databases-anywhere-with-ibm-cloud-satellite).

## 5 May 2021
{: #databases-for-postgresql-05may2021}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} 9.6 End of Life
:  On 11 November 2021, all {{site.data.keyword.cloud_notm}} {{site.data.keyword.databases-for-postgresql}} instances on version 9.6 that are still active will be disabled. See blog post announcement [here](https://www.ibm.com/cloud/blog/announcements/ibm-cloud-databases-postgresql-9-6-end-of-life).

## 15 February 2021
{: #databases-for-postgresql-15feb2021}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} Horizontal Scaling
:  Customers can scale {{site.data.keyword.databases-for-postgresql_full}} by adding members to their database instance. See blog post announcement [here](https://www.ibm.com/cloud/blog/announcements/whats-new-in-ibm-cloud-databases).

## 9 Sept 2020
{: #databases-for-postgresql-09sep2020}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} 9.5 End of Life
:  On November 11, 2021, all {{site.data.keyword.cloud_notm}} {{site.data.keyword.databases-for-postgresql}} instances on version 9.5 that are still active will be disabled. See blog post announcement [here](https://www.ibm.com/cloud/blog/announcements/postgresql-9-5-end-of-life).

## 13 April 2020
{: #databases-for-postgresql-13apr2020}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} autoscaling
:  Starting after 26 April 2020, all deployments of {{site.data.keyword.databases-for-postgresql_full}} will have a minor change in the format of PostgreSQL logs emitted into Log Analysis with LogDNA.  See blog post announcement [here](https://www.ibm.com/cloud/blog/announcements/ibm-cloud-databases-portfolio-introduces-autoscaling).

## 27 March 2020
{: #databases-for-postgresql-27mar2020}
{: release-note}

Changes to {{site.data.keyword.databases-for-postgresql_full}} Logging
:  We are excited to announce that autoscaling of your deployments based on disk capacity and disk I/O utilization is now available for {{site.data.keyword.databases-for-postgresql_full}}via the UI, API, and CLI. See blog post announcement [here](https://www.ibm.com/cloud/blog/announcements/changes-to-databases-for-postgresql-logging).

## 2 October 2019
{: #databases-for-postgresql-02oct2019}
{: release-note}

{{site.data.keyword.databases-for-postgresql_full}} Announces Point-In-Time Recovery
:  {{site.data.keyword.databases-for-postgresql_full}} now allows you to restore a database into a new instance from a specific timestamp. See blog post announcement [here](https://www.ibm.com/cloud/blog/announcements/ibm-cloud-databases-for-postgresql-announces-point-in-time-recovery).

## 6 August 2019
{: #databases-for-postgresql-06aug2019}
{: release-note}

New Regions Available for {{site.data.keyword.cloud_notm}} Database Services
:  {{site.data.keyword.databases-for-postgresql_full}} is now available to be deployed in Seoul; South Korea; and Chennai, India. See blog post announcement [here](https://www.ibm.com/cloud/blog/announcements/new-regions-available-for-ibm-cloud-database-services).

## 2 October 2018
{: #databases-for-postgresql-02oct2018}
{: release-note}

General Availability of {{site.data.keyword.databases-for-postgresql_full}}
:  {{site.data.keyword.databases-for-postgresql_full}} added to the [{{site.data.keyword.cloud_notm}} Databases](https://www.ibm.com/cloud/databases) family. See blog post announcement [here](https://www.ibm.com/cloud/blog/ibm-cloud-databases-for-postgresql-and-databases-for-redis-are-now-generally-available).
