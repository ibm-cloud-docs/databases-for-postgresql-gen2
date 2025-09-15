---
copyright:
  years: 2020, 2025
lastupdated: "2025-09-15"

keywords: postgresql, databases, upgrading, major versions, postgresql new deployment, postgresql database version, postgresql major version

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Upgrading to a new major version
{: #upgrading}

When a major version of a database is nearing its End Of Life (EOL), it is advisable to upgrade to a current major version. 

Find the available versions of {{site.data.keyword.databases-for-postgresql}} in the [{{site.data.keyword.cloud_notm}} catalog](https://cloud.ibm.com/databases/databases-for-postgresql/create) page, from the {{site.data.keyword.databases-for}} CLI plug-in command [`ibmcloud cdb deployables-show`](/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference#deployables-show){: external}, or from the {{site.data.keyword.databases-for}} API [`/deployables`](/apidocs/cloud-databases-api/cloud-databases-api-v5#listdeployables){: external} endpoint.

When you upgrade to a new instance, you also need to change the connection information in your application.
{: note}

## Requirements for upgrading to newer PostgreSQL major version from PostgreSQL v13 
{: #upgrading-reqs}

If you have `pg_repack` installed, you need to remove it before performing the upgrade. This can be done with a command like:

```sh
DROP EXTENSION pg_repack; 
```
{: pre}

After upgrading, reinstall `pg_repack`. This can be done with the following command:

```sh
CREATE EXTENSION pg_repack;
```
{: pre}

If you are using PostGIS, you must first upgrade to PostGIS before upgrading PostgreSQL. This can be done by running the following command against a database with PostGIS installed.
{: note}

```sh
SELECT postgis_extensions_upgrade();
```
{: pre}

Use the following query to validate the PostGIS extension upgrade. 

```sh
SELECT postgis_full_version();
```
{: pre}


## Upgrading from a read-only replica
{: #upgrading-replica}

Upgrade by [configuring a read-only replica](/docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas). Provision a read-only replica with the same database version as your deployment and wait while it replicates all of your data. When your deployment and its replica are synced, promote and upgrade the read-only replica to a full, stand-alone deployment running the new version of the database. To perform the upgrade and promotion step, use a POST request to the [`/deployments/{id}/remotes/promotion`](/apidocs/cloud-databases-api/cloud-databases-api-v5#promotereadonlyreplica) endpoint with the version that you want to upgrade to in the body of the request. 

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

To evaluate the effects of major version upgrades, trigger a dry run. A dry run simulates the promotion and upgrade, with the results printed to the database logs. Access and view your database logs through the [Log analysis integration](/docs/cloud-databases?topic=cloud-databases-logging). This ensures the version that you are currently running with its extensions can be successfully upgraded to your intended version.

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

You can upgrade your database version by [restoring a backup](/docs/cloud-databases?topic=cloud-databases-dashboard-backups&interface=ui#restore-backup-ui) of your data into a new deployment that is running the new database version.

### Upgrading in the UI
{: #upgrading-ui}
{: ui}

Upgrade to a new version when [restoring a backup](/docs/cloud-databases?topic=cloud-databases-dashboard-backups&interface=ui#restore-backup-ui) from the _Backups_ menu of your _Deployment dashboard_. Click **Restore** on a backup to the provisioning page on a new tab, where you can change some options for the new deployment. One of the options is the database version, which is auto-populated with the versions available for you to upgrade to. Select a version and click **Create** to start the provision and restore process.

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

Complete the necessary steps to use the [Resource controller API](/docs/databases-for-postgresql?topic=databases-for-postgresql-provisioning&interface=api#provision-controller-api) before you use it to upgrade from a backup. Then, send the API a `POST` request. The parameters `name`, `target`, `resource_group`, and `resource_plan_id` are all required. You also supply the version and backup ID. The new deployment has the same memory and disk allocation as the source deployment at the time of the backup. 

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

After the end-of-life date, all active {{site.data.keyword.databases-for-postgresql}} deployments on the deprecated version will be forcibly upgraded to the next supported version. For example, PostgreSQL Version 13 (deprecated) upgrades to Version 14.
{: .note}

**Upgrade before the end-of-life date to avoid the following risks:**

- No SLAs are provided for this type of forced upgrade.
- You may experience some data loss.
- Your application may experience prolonged downtime.
- Your application may stop working if it is incompatible with the new version.
- You cannot control the timing of when this upgrade will happen for your deployment.
- There is no rollback process for this forced upgrade.

For the end-of-life dates, refer to the [version policy page](https://cloud.ibm.com/docs/cloud-databases?topic=cloud-databases-versioning-policy ){: external}.


## _Admin user_ issues during version upgrades
{: #admin_user_issues}

Starting with PostgreSQL 16, role privilege enforcement has become more stringent. In contrast, earlier versions allowed roles with the `CREATEROLE` attribute to manage other roles; version 16 and beyond require that a role has the `ADMIN OPTION` on another role to grant or revoke it.
{: .note}

**If you are upgrading your database from PostgreSQL v15 or earlier to PostgreSQL v16 or higher, you may encounter privilege-related errors, such as:**

```sh
ERROR: only roles with the ADMIN OPTION on role "some_role" may grant this role
DETAIL: role "admin" is not permitted to grant role "some_role"
```

To address this issue, we provide a built-in helper function `grant_admin_option_to_roles` that is designed to restore the expected role management behavior.

This function:

- Applies only to databases upgraded from PostgreSQL v15 and earlier versions to PostgreSQL 16 and later (if you're encountering the error described above).
- Accepts an arbitrary list of roles to apply the fix to.
- Can only be executed by the `admin user`.
- Is safe to run multiple times (idempotent).

Sample usage:

```
SELECT grant_admin_option_to_roles('role1', 'role2', 'role3');
```

This ensures that the `admin user` retains the appropriate `ADMIN OPTION` privileges to manage designated roles in upgraded instances.

## Changelog for major PostgreSQL versions
{: #changelog-postgres}

- [PostgreSQL 13](https://www.postgresql.org/docs/13/release-13.html){: external}
- [PostgreSQL 14](https://www.postgresql.org/docs/14/release-14.html){: external}
- [PostgreSQL 15](https://www.postgresql.org/docs/release/15.0/){: external}
- [PostgreSQL 16](https://www.postgresql.org/docs/release/16.0/){: external}
- [PostgreSQL 17](https://www.postgresql.org/docs/release/17.0/){: external}
