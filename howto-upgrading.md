---
copyright:
  years: 2020, 2026
lastupdated: "2026-08-27"

keywords: postgresql, databases, upgrading, major versions, postgresql new deployment, postgresql database version, postgresql major version

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Upgrading to a new major version
{: #upgrading}

{{site.data.keyword.databases-for-postgresql}} provides three different upgrade paths:

- In-place upgrade to a new major version.
- Restoring from backup.
- Upgrading from a read-only replica.

When a major version of a database is nearing its End Of Life (EOL), it is advisable to upgrade to a current major version.

Find the available versions of {{site.data.keyword.databases-for-postgresql}} in the [{{site.data.keyword.cloud_notm}} catalog](https://cloud.ibm.com/catalog) page, from the {{site.data.keyword.databases-for}} CLI plug-in command [`ibmcloud cdb deployables-show`](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-cdb-reference){: external}, or from the {{site.data.keyword.databases-for}} API [`/deployables`](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-api){: external} endpoint.

When you upgrade to a new instance, you also need to change the connection information in your application.
{: note}

In the following example commands, the full CRN of the database instance is required for the {id}. Since the CRN contains special characters, it must be *URL-encoded* to avoid a "not_found" error.
{: note}


## Requirements for upgrading to a newer PostgreSQL major version
{: #upgrading-reqs}

Before you start any major version upgrade path, review any extensions, replication objects, and application dependencies that must be maintained first.

Some extensions and logical replication objects are version-specific or depend on server-side components that must match the PostgreSQL major version. Removing them before the upgrade helps avoid failures and lets you re-create only the supported objects after the new version is running.

### Extensions and logical replication objects to review
{: #review-extensions}

Review the following items before the upgrade:

**Extensions**
- `pg_repack`
- `old_snapshot`
- `wal2json`
- `anon`
- `PostGIS`

**Replication slots**
- `Logical replication slots`

**Application dependencies**
If you remove extensions or replication objects that your applications depend on, validate your data flows and application behavior before you proceed with the upgrade. Also consider the possible disruptions in your application logic that depends on specific PostgreSQL features.
{: note}

### `pg_repack`
{: #pg_repack}

Drop `pg_repack` before the upgrade and re-create it after the upgrade. `pg_repack` uses a version-specific extension and client/server components that must match the PostgreSQL major version.

```sh
DROP EXTENSION pg_repack;
```
{: pre}

Re-create the extension after the upgrade only if your workload still requires it.

```sh
CREATE EXTENSION pg_repack;
```
{: pre}

### `old_snapshot`
{: #old_snapshot}

Drop `old_snapshot` before the upgrade. **Do not** re-create it after upgrading to PostgreSQL 18 because it is no longer supported.

```sh
DROP EXTENSION old_snapshot;
```
{: pre}

### `wal2json` replication slots
{: #wal2json}

If you use `wal2json` for logical decoding, you must drop all associated replication slots before the upgrade. The `pg_upgrade` utility strictly forbids major version upgrades while replication slots exist and will throw a critical error and halt the upgrade.

Before upgrading:

1. Ensure all pending WAL data has been consumed.
2. Stop your application that uses the replication slot.
3. Drop the replication slot(s):

```sh
SELECT pg_drop_replication_slot('your_slot_name');
```
{: pre}

After the upgrade, you can recreate the replication slots as needed. Note that `wal2json` is not installed via `CREATE EXTENSION` but is configured through database parameters (`wal_level`, `max_replication_slots`, `max_wal_senders`) and table permissions, which do not prevent upgrades.

### `anon`
{: #upgrading-handling}

Remove the `anon` extension before the upgrade and re-enable it after the upgrade if you still need it. Additional steps are required before you drop `anon`.

If the `anon` extension is installed, follow the steps below and execute the commands as the admin user before performing the upgrade.

1. Remove all masking rules (if enabled).

    ```sh
    SELECT anon.remove_masks_for_all_columns();
    ```
    {: pre}

2. Disable masking roles (the upgrade might fail if any roles are marked as masked).

    ```sh
    SECURITY LABEL FOR anon ON ROLE <role_name> IS NULL;
    ```
    {: pre}

3. Drop the `anon` extension with the cascade option.

    ```sh
    DROP EXTENSION anon CASCADE;
    ```
    {: pre}

4. If the `anon` extension is installed in multiple databases within an instance, follow the outlined steps for each database.

5. After the upgrade is complete, re-enable the `anon` extension and reapply masking rules as required.

It is strongly recommended to validate the data both before and after dropping the extension to ensure masking consistency prior to performing the upgrade.
{: note}

### `PostGIS`
{: #postgis}

If you are using PostGIS, first upgrade PostGIS before you upgrade PostgreSQL.

```sh
SELECT postgis_extensions_upgrade();
```
{: pre}

Use the following query to validate the PostGIS extension upgrade.

```sh
SELECT postgis_full_version();
```
{: pre}

### `Logical replication slots`
{: #logical-replication}

Drop all logical replication slots before the upgrade and re-create them after the upgrade. Logical slots are tied to the source server state and should be re-created cleanly on the upgraded instance.

```sh
SELECT pg_drop_replication_slot('<slot_name>');
```
{: pre}

## In-place major version upgrades
{: #upgrading-in-place}

An in-place major version upgrade (IPU) allows you to upgrade your deployment to a supported target /docs/databases-for-postgresql?topic=databases-for-postgresql-versioning-policy#version-definitions without restoring a backup into a new deployment. The upgrade keeps existing connection strings, so no reconfiguration is required.

However, application changes might be required if the new version introduces compatibility differences.

During the upgrade window, your deployment experiences a brief downtime. The duration depends on the size and complexity of your deployment.

If your applications must continue to read data during the upgrade, you can provision a /docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas&interface=ui#read-only-replicas-provision and update your application to use the replica. You can promote the replica to primary if the upgrade does not complete successfully. For more information, see /docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas&interface=ui#read-only-replicas-ipu.
{: important}

{{site.data.keyword.databases-for-postgresql}} does not automatically create backups before or after an in-place major version upgrade.

To improve recoverability, create:

- A backup before the upgrade to protect your current data state
- A backup immediately after the upgrade to establish the first restore point for the new version

If you do not take a backup after the upgrade, point-in-time recovery (PITR) is not available for the new version until the next scheduled backup completes.

Backups and PITR points created before the upgrade remain associated with the earlier version and cannot be restored into the upgraded version. However, they can still be used to restore the earlier version into a new deployment.
{: important}

### Logical replication slots
{: #logical-replication}

Drop all logical replication slots before the upgrade, and re-create them afterward. Logical replication slots are tied to the source server state and must be re-created on the upgraded instance.

```sh
SELECT pg_drop_replication_slot('<slot_name>');
```
{: pre}

### Before you begin
{: #upgrading-considerations}

Review the following before you start the upgrade:

- Verify that version upgrades are supported for your deployment by using the UI, API, CLI, or Terraform.

  Example (CLI):

  ```bash
  ibmcloud cdb capability-show versions postgresql
  ```
  {: pre}

- Review precheck requirements. The upgrade runs on the source deployment and is blocked
if risks are detected. Ensure that:

  - The deployment is healthy
  - At least 10% free disk space is available
  - I/O utilization is below 90%
  - Schema size and object counts are within supported limits
  - Required extension and logical replication slot cleanup is complete

- Review the https://www.postgresql.org/docs/release/ for compatibility changes that might affect your applications.
- Downgrading to an earlier version is not supported.
- An in-place upgrade cannot be canceled after it starts.
- Ensure that a recent backup is available before you upgrade.

| Source PostgreSQL version | Supported in-place upgrade target |
|--------------------------|-----------------------------------|
| 18                       | Future major versions (when available) |
{: caption="Supported in-place upgrade paths for Gen2" caption-side="bottom"}

Gen2 starts with PostgreSQL 18. Upgrade paths to newer versions are added as they become supported. For earlier versions (14–17), see the /docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading.
{: .note}

After the upgrade completes, your deployment runs a new PostgreSQL major version. Backups and PITR points from before the upgrade belong to the earlier version timeline and cannot be restored into the upgraded version.

To maintain restore and PITR capabilities for the new version, take a backup immediately after the upgrade. This backup becomes the baseline for future recovery operations.

If the upgrade fails, backups from before the upgrade can still be used with PITR to restore the earlier version into a new deployment.
{: important}

### Upgrading in the UI
{: #upgrading-in-place-ui}
{: ui}

1. Create a test deployment by restoring a backup from your existing deployment with the same version.
2. Update your staging application to use the test deployment and validate functionality.
3. Start the upgrade from the **Overview** page by clicking **Upgrade major version**.
4. Validate application behavior on the upgraded test deployment.
5. Upgrade your production deployment after validation completes.

   After the upgrade starts, it cannot be stopped or rolled back. Ensure that a recent backup is available.

The expiration for starting upgrade defines how long the upgrade job has to begin before it is automatically canceled. Set this value based on your maintenance window. For example, if the upgrade takes 30 minutes and your window is 1 hour, set the expiration to 30 minutes. The expiration can be between 5 minutes and 24 hours.

### Upgrading through the API
{: #upgrading-in-place-api}
{: api}

Use the following request to start an in-place upgrade:

```sh
curl -X PATCH https://api.{region}.databases.cloud.ibm.com/v5/ibm/deployments/{id}/version \
  -H 'Authorization: Bearer <>' \
  -H 'Content-Type: application/json' \
  -d '{"version": "15"}'
```
{: pre}

For more information, see the [Cloud Databases API](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5#setdatabaseinplaceversionupgrade).

### Upgrading through the CLI
{: #upgrading-in-place-cli}
{: cli}

Available in CDB plugin version >= 0.20.0.
{: .note}

To view available upgrade paths:

```sh
ibmcloud cdb deployment-capability-show <NAME|CRN> versions
```

To start an upgrade:

```sh
ibmcloud cdb deployment-version-upgrade <NAME|CRN> <TARGET_VERSION>
```

For command details:

```sh
ibmcloud cdb deployment-version-upgrade --help
```

Use either `--expire-in` or `--expire-at` to set the expiration time.

### Upgrading through Terraform
{: #upgrading-in-place-terraform}
{: terraform}

Available in Terraform provider version >= 1.79.2.
{: .note}

To upgrade, update the `version` value in your configuration.

Skipping a backup before an upgrade can result in data loss if the upgrade fails. Ensure that a recent backup is available.
{: .attention}

Increase the timeout if needed, as Terraform uses timeouts instead of expiration timestamps.

### Troubleshooting
{: #upgrading-in-place-troubleshooting}

If issues occur after a successful upgrade and you need to return to the previous version, contact {{site.data.keyword.cloud}} support for guidance. Avoid performing PITR or restores without guidance, as this can complicate recovery.
{: .attention}

Upgrades run only after all prechecks pass. If the upgrade is blocked, verify:

- Cluster health (Patroni state is stable)
- Sufficient free disk space
- Acceptable disk I/O utilization
- Schema size and object count limits

Large schemas and high object counts can increase upgrade duration.

If upgrade attempts continue to fail, open a support ticket through https://cloud.ibm.com/login?redirect=%2Funifiedsupport%2Fsupportcenter.

## Upgrading from a read-only replica
{: #upgrading-replica}

Upgrade by [configuring a read-only replica](/docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas). Provision a read-only replica with the same database version as your deployment and wait while it replicates all of your data. When your deployment and its replica are synced, promote and upgrade the read-only replica to a full, stand-alone deployment running the new version of the database. To perform the upgrade and promotion step, use a POST request to the [`/deployments/{id}/remotes/promotion`](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-api) endpoint with the version that you want to upgrade to in the body of the request.

This request looks like:

```sh
curl -X POST \
  https://api.{region}.databases.cloud.ibm.com/v5/ibm/deployments/{id}/remotes/promotion \
  -H 'Authorization: Bearer <>'  \
 -H 'Content-Type: application/json' \
 -d '{
    "promotion": {
        "version": "14",
        "skip_initial_backup": false
    }
}' \
```
{: pre}

`skip_initial_backup` is optional. If set to `true`, the new deployment does not take an initial backup when the promotion completes. Your new deployment is available in a shorter amount of time, at the expense of not being backed up until the next automatic backup is run, or you take an on-demand backup.


### Dry running the promotion and upgrade
{: #promotion-dry-run}

To evaluate the effects of major version upgrades, trigger a dry run. A dry run simulates the promotion and upgrade, with the results printed to the database logs. Access and view your database logs through the [Log analysis integration](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-logging). This ensures the version that you are currently running with its extensions can be successfully upgraded to your intended version.

The dry run must be run with `skip_initial_backup` set to `false`, and `version` defined.

The command looks like:

```sh
curl -X POST \
  https://api.{region}.databases.cloud.ibm.com/v5/ibm/deployments/{id}/remotes/promotion \
  -H 'Authorization: Bearer <>'  \
 -H 'Content-Type: application/json' \
 -d '{
    "promotion": {
        "version": "14",
        "skip_initial_backup": false,
        "dry_run": true
    }
}' \
```
{: pre}

## Back up and restore upgrade
{: #backup-restore}

You can upgrade your database version by [restoring a backup](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-dashboard-backups&interface=ui) of your data into a new deployment that is running the new database version.

### Upgrading in the UI
{: #upgrading-ui}
{: ui}

Upgrade to a new version when [restoring a backup](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-dashboard-backups&interface=ui#restore-backup) from the _Backups_ menu of your _Deployment dashboard_. Click **Restore** on a backup to the provisioning page on a new tab, where you can change some options for the new deployment. One of the options is the database version, which is auto-populated with the versions available for you to upgrade to. Select a version and click **Create** to start the provision and restore process.

### Upgrading through the CLI
{: #upgrading-cli}
{: cli}

To upgrade and restore from a backup through the {{site.data.keyword.cloud_notm}} CLI, use the provisioning command from the resource controller.

```sh
ibmcloud resource service-instance-create <DEPLOYMENT_NAME_OR_CRN> <SERVICE_ID> <SERVICE_PLAN_ID> <REGION>
```
{: pre}

The parameters `service-name`, `service-id`, `service-plan-id`, and `region` are all required. You also supply the `-p` with the version and backup ID parameters in a JSON object. The new deployment is automatically sized with the same disk and memory as the source deployment at the time of the backup.

This command looks like:

```sh
ibmcloud resource service-instance-create example-upgrade databases-for-postgresql standard us-south \
-p \ '{
  "backup_id": "crn:v1:bluemix:public:databases-for-postgresql:us-south:a/54e8ffe85dcedf470db5b5ee6ac4a8d8:1b8f53db-fc2d-4e24-8470-f82b15c71717:backup:06392e97-df90-46d8-98e8-cb67e9e0a8e6",
  "version":14
}'
```
{: pre}

### Upgrading through the API
{: #upgrading-api}
{: api}

Complete the necessary steps to use the [Resource controller API](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-provisioning&interface=api) before you use it to upgrade from a backup. Then, send the API a `POST` request. The parameters `name`, `target`, `resource_group`, and `resource_plan_id` are all required. You also supply the version and backup ID. The new deployment has the same memory and disk allocation as the source deployment at the time of the backup.

This command looks like:

```sh
curl -X POST \
  https://resource-controller.cloud.ibm.com/v2/resource_instances \
  -H 'Authorization: Bearer <>' \
  -H 'Content-Type: application/json' \
    -d '{
    "name": "my-instance",
    "target": "bluemix-us-south",
    "resource_group": "5g9f447903254bb58972a2f3f5a4c711",
    "resource_plan_id": "databases-for-postgresql-standard",
    "backup_id": "crn:v1:bluemix:public:databases-for-postgresql:us-south:a/54e8ffe85dcedf470db5b5ee6ac4a8d8:1b8f53db-fc2d-4e24-8470-f82b15c71717:backup:06392e97-df90-46d8-98e8-cb67e9e0a8e6",
    "version":14
  }'
```
{: pre}

## Forced upgrade
{: #forced_upgrade}

After the end-of-life date, all active {{site.data.keyword.databases-for-postgresql}} deployments that run a deprecated version are automatically upgraded to the next supported version. For example, PostgreSQL 13 (deprecated) is upgraded to version 14.
{: .note}

**Upgrade before the end-of-life date to avoid the following risks:**

- No SLAs are provided for this type of forced upgrade.
- You may experience some data loss.
- Your application may experience prolonged downtime.
- Your application may stop working if it is incompatible with the new version.
- You cannot control the timing of when this upgrade will happen for your deployment.
- There is no rollback process for this forced upgrade.

For the end-of-life dates, refer to the [version policy page](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-versioning-policy){: external}.

## _Role privilege_ issues during version upgrades
{: #_role_privilege_issues}

Starting with PostgreSQL 16, role privilege enforcement is more stringent. This is an upstream PostgreSQL architectural change, not a behavior change specific to {{site.data.keyword.IBM}}. In earlier versions, roles with the `CREATEROLE` attribute could manage other roles more broadly. In PostgreSQL 16 and later, a role must have the `ADMIN OPTION` on another role to grant or revoke it. For more information, see the PostgreSQL 16 [release notes](https://www.postgresql.org/docs/16/release-16.html){: external}, [role attributes](https://www.postgresql.org/docs/16/role-attributes.html){: external}, and [`GRANT` on roles](https://www.postgresql.org/docs/16/sql-grant.html){: external}.
{: .note}

If you are upgrading from PostgreSQL 15 or earlier to PostgreSQL 16 or later, review your role grants before starting in-place upgrade (IPU). If role management must continue after the upgrade, ensure that the required roles are granted with WITH ADMIN OPTION before you start the upgrade.
{: warning}

**If you encounter privilege-related errors after the upgrade, for example:**

```sh
ERROR: only roles with the ADMIN OPTION on role "some_role" may grant this role
DETAIL: role "admin" is not permitted to grant role "some_role"
```
{: pre}

**Use the built-in helper function `grant_admin_option_to_roles` to restore `ADMIN OPTION` for specific roles:**

- Applies only to databases upgraded from PostgreSQL v15 and earlier versions to PostgreSQL 16 and later (if you're encountering the error described above).
- Accepts an arbitrary list of roles to apply the fix to.
- Can only be executed by the `admin user`.
- Is safe to run multiple times (idempotent).

Sample usage:

```sh
SELECT grant_admin_option_to_roles('role1', 'role2', 'role3');
```
{: pre}

This function grants the specified roles (`role1`, `role2`, `role3`) to the `admin` user with `ADMIN OPTION`, allowing the `admin` user to manage (grant, revoke, alter, or drop) these roles in upgraded instances.

## Changelog for major PostgreSQL versions
{: #changelog-postgres}

- [PostgreSQL 18](https://www.postgresql.org/docs/release/18.0/){: external}

For information about earlier PostgreSQL versions (14-17), see the [Gen1 changelog](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading#changelog-postgres).
