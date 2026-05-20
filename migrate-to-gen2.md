---
copyright:
  years: 2026
lastupdated: "2026-05-20"

keywords: postgresql migration, gen1 to gen2, classic infrastructure, vpc, read replica, backup and restore, private endpoints, version upgrade

subcollection: databases-for-postgresql-gen2

---

{{site.data.keyword.attribute-definition-list}}

# Migrating {{site.data.keyword.databases-for-postgresql}} from Gen 1 to Gen 2

This tutorial shows you how to migrate your {{site.data.keyword.databases-for-postgresql}} deployment from Gen 1 on Classic Infrastructure to Gen 2 on VPC.
{: shortdesc}

Use this tutorial to choose a migration path, prepare your network, and complete either a read replica migration for minimal downtime or a backup and restore migration during a planned maintenance window.

Migration from Gen 1 to Gen 2 changes the networking model for your deployment. Gen 2 deployments support private endpoints only. 
{: important}

Follow these steps to complete a read replica migration: 

* [Before you begin](#prereqs)
* [Step 1: Review the available migration paths](#migration-paths)
* [Step 2: Confirm version and network readiness](#readiness)
* [Step 3: Upgrade your Gen 1 deployment to PostgreSQL 18](#upgrade-gen1)
* [Step 4: Provision the Gen 2 read replica](#provision-replica)
* [Step 5: Validate replication and prepare for cutover](#validate-replica)
* [Step 6: Cut over to Gen 2](#cutover)
* [Step 7: Complete post-migration tasks](#post-migration)
{: ui}

Follow these steps to complete a backup and restore migration:

* [Before you begin](#prereqs)
* [Step 1: Review the available migration paths](#migration-paths)
* [Step 2: Confirm version and network readiness](#readiness)
* [Step 3: Upgrade your Gen 1 deployment to PostgreSQL 18](#upgrade-gen1)
* [Step 4: Create a backup of your Gen 1 deployment](#backup-restore)
* [Step 5: Restore the backup to a new Gen 2 deployment](#restore-gen2)
* [Step 6: Update your applications and validate the migration](#validate-backup-restore)
* [Step 7: Complete post-migration tasks](#post-migration)
{: ui}

## Before you begin
{: #prereqs}

Make sure that you have the following prerequisites in place:

* A Gen 1 {{site.data.keyword.databases-for-postgresql}} deployment that you want to migrate.
* Access to provision a Gen 2 {{site.data.keyword.databases-for-postgresql}} deployment in your target account, resource group, and region.
* A maintenance window that is approved for your migration activity.
* Access to the application configuration so that you can update connection information during cutover.

Complete these checks before you start the migration:

* Verify your current PostgreSQL version.
* Verify that no in-progress operations, such as scaling, backups, or upgrades, are running on the source deployment.
* Verify that you have a recent successful backup.
* Verify disk utilization and available capacity.
* Identify whether your applications use public endpoints, private endpoints, or both.

If your applications use public endpoints today, plan your network changes before you start the data migration. Gen 2 supports private endpoints only.
{: note}

## Step 1: Review the available migration paths
{: #migration-paths}

You can migrate your {{site.data.keyword.databases-for-postgresql}} deployment by using one of the following approaches:

### Read replica migration
{: #read-replica-path}

Use a read replica migration when you want the lowest possible downtime. This approach is the recommended migration path for PostgreSQL.

The high-level flow is as follows:

1. Upgrade the Gen 1 deployment to PostgreSQL 18 if it is not already on version 18.
1. Provision a Gen 2 read replica from the Gen 1 primary.
1. Allow replication to catch up.
1. Validate the Gen 2 deployment.
1. Promote the Gen 2 replica to become the new primary.
1. Cut over your applications to Gen 2.

This approach gives you time to validate the target deployment before cutover and reduces downtime to the final switchover event.
{: tip}

### Backup and restore migration
{: #backup-restore-path}

Use backup and restore when a planned downtime window is acceptable or when a replica-based migration is not the best fit for your environment.

The high-level flow is as follows:

1. Upgrade the Gen 1 deployment to PostgreSQL 18 if it is not already on version 18.
1. Put the Gen 1 deployment into read-only mode.
1. Create a backup of the Gen 1 deployment.
1. Restore the backup into a new Gen 2 deployment.
1. Update your applications to connect to the Gen 2 deployment.
1. Validate the migrated data and application behavior.

Backup and restore requires downtime for the migration window because your applications must stop writing to the source deployment before the final backup is taken, and remain stopped until the restored Gen 2 deployment is ready to serve traffic.
{: note}

## Step 2: Confirm version and network readiness
{: #readiness}

At launch, Gen 2 supports PostgreSQL 18 for this migration flow.

Review the supported paths:

| Source version on Gen 1 | Required Gen 1 action | Target version on Gen 2 | Available migration paths |
| --- | --- | --- | --- |
| 15 | Upgrade to PostgreSQL 18 | 18 | Read replica migration or backup and restore |
| 16 | Upgrade to PostgreSQL 18 | 18 | Read replica migration or backup and restore |
| 17 | Upgrade to PostgreSQL 18 | 18 | Read replica migration or backup and restore |
| 18 | No version upgrade required | 18 | Read replica migration or backup and restore |

If your source deployment is not already on PostgreSQL 18, upgrading on Gen 1 is a mandatory prerequisite for both migration paths.
{: important}

Also confirm that your target networking is ready:

* Set up your VPC infrastructure.
* Configure the required private connectivity, such as VPE or VPN access, based on your application architecture.
* Confirm that your applications can reach private service endpoints.
* Prepare to update connection strings, certificates, and any allowlists that depend on endpoint addresses.

Before you migrate a production deployment, validate private connectivity by using a test Gen 2 deployment or by rehearsing the migration with a development or test instance.
{: tip}

## Step 3: Upgrade your Gen 1 deployment to PostgreSQL 18
{: #upgrade-gen1}

PostgreSQL 18 is required on Gen 1 before you can migrate to Gen 2. If your deployment runs PostgreSQL 15, 16, or 17, complete the required in-place upgrade on Gen 1 before you begin either the read replica or backup and restore migration path.

If your deployment already runs PostgreSQL 18, you can continue with your selected migration path.

Upgrading to PostgreSQL 18 on Gen 1 can introduce its own downtime event. If your deployment requires this upgrade, plan for two service-impacting events:

1. The in-place upgrade to PostgreSQL 18 on Gen 1.
1. The final cutover from Gen 1 to Gen 2.

For upgrade instructions, see the [in-place upgrade documentation](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading-postgresql#upgrading-postgresql-in-place).

## Step 4: Provision the Gen 2 read replica
{: #provision-replica}

Provision a Gen 2 deployment as a read replica of the Gen 1 primary.

When you provision the replica, review the following items:

* Region, resource group, and tags for the new deployment.
* Compute profile sizing for the target deployment.
* Disk sizing that meets or exceeds the source deployment requirements.
* Encryption settings, including any customer-managed key requirements.

Make sure that the replica is created with enough capacity to handle production traffic after promotion.
{: note}

If you are testing the migration for the first time, complete the full procedure in a non-production environment before you schedule the production cutover.

## Step 5: Validate replication and prepare for cutover
{: #validate-replica}

After the Gen 2 replica is provisioned, monitor replication until the replica is fully synchronized with the Gen 1 primary.

Review the following checks before you continue:

* Replica deployment status is healthy.
* Replication lag is within an acceptable threshold for your workload.
* The target deployment is reachable through the planned private network path.
* Monitoring and logging integrations are configured as needed.
* Application owners are ready to update connection information during cutover.

Use the validation period to confirm that the Gen 2 deployment meets your operational requirements, including capacity, observability, and network connectivity.

Do not begin cutover until replication health is stable and your rollback plan is documented.
{: important}

## Step 6: Cut over to Gen 2
{: #cutover}

During the migration window, complete the cutover in a controlled sequence.

1. Put the Gen 1 primary into a read-only state that prevents new writes.
1. Confirm that the Gen 2 replica is fully synchronized.
1. Promote the Gen 2 replica to become the primary deployment.
1. Update application connection strings and related configuration to point to the Gen 2 deployment.
1. Validate that applications can connect and perform read and write operations successfully.

The cutover event is the primary downtime event for the read replica migration path.
{: note}

After promotion, confirm that the Gen 2 deployment is operating independently and that the Gen 1 deployment is no longer part of the active production path.

## Step 7: Complete post-migration tasks
{: #post-migration}

After the migration is complete:

* Validate application behavior and key business workflows.
* Review performance, logs, and monitoring dashboards.
* Confirm backup policies and alerting on the Gen 2 deployment.
* Update runbooks, inventory records, and operational documentation.
* Keep the Gen 1 deployment available for your planned retention period, if required by your rollback policy.
* Decommission the Gen 1 deployment only after you complete your validation and retention requirements.

## Create a backup of your Gen 1 deployment
{: #backup-restore}

Use this path only if you selected backup and restore as your migration method.

Before you take the migration backup:

* Confirm that the source deployment is running PostgreSQL 18.
* Put the Gen 1 deployment into read-only mode so that no new writes occur during backup creation and restore preparation.
* Verify that the backup completed successfully and is available for restore.

After the backup is taken, you can keep the Gen 1 deployment in read-only mode until the migration is validated, or return it to read/write mode later if you need to delay cutover or use it as part of your fallback plan.

Because this path relies on a backup image, the duration of downtime depends on the size of your data and the time that is required to restore and validate the new deployment.
{: important}

## Restore the backup to Gen 2
{: #restore-gen2}

Restore the selected backup into a new Gen 2 {{site.data.keyword.databases-for-postgresql}} deployment.

As part of the restore workflow, verify the following items:

* The target deployment uses the correct sizing and storage configuration.
* The restore operation completes successfully.
* Private connectivity to the Gen 2 deployment is available before you reopen application traffic.

## Update your applications and validate the migration
{: #validate-backup-restore}

After the restore completes:

1. Update application connection information to use the Gen 2 deployment.
1. Validate that the required schemas, databases, and roles are present.
1. Run application smoke tests and data validation checks.
1. Resume production traffic only after validation completes successfully.

## Planning considerations
{: #troubleshooting}

Keep the following migration considerations in mind:

* Customers who are already on PostgreSQL 18 can typically complete a single migration event for read replica cutover.
* Customers on PostgreSQL 15, 16, or 17 must first upgrade on Gen 1, which adds a separate service-impacting event.
* Network preparation is a required part of the migration because Gen 2 supports private endpoints only.
* Replica lag, active connections, and backup readiness are key health indicators during migration planning.

If you encounter issues during migration, pause before cutover and validate source health, network connectivity, version readiness, and replication status.

## Next steps
{: #next-steps}

* Review your service-specific networking requirements for private endpoint access.
* Update your internal operating procedures for Gen 2 connectivity, monitoring, and backup management.
* Repeat the migration in a test environment to refine your production runbook.
* Decommission the Gen 1 deployment only after you complete post-migration validation and retention requirements.
