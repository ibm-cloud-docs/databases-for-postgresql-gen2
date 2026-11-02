---
copyright:
  years: 2026
lastupdated: "2026-11-02"

keywords: postgresql migration, gen1 to gen2, classic infrastructure, vpc, read replica, backup and restore, terraform, ibmcloud cli, private endpoints, version upgrade, context-based restrictions

subcollection: databases-for-postgresql-gen2

---

{{site.data.keyword.attribute-definition-list}}

# Migrating {{site.data.keyword.databases-for-postgresql}} from Gen 1 to Gen 2
{: #migrating-postgresql-gen1-to-gen2}

[Gen 2]{: tag-purple}

Use this tutorial to migrate your {{site.data.keyword.databases-for-postgresql}} deployment from Gen 1 (Classic) to Gen 2 (VPC) by using the Migration hub in the {{site.data.keyword.cloud_notm}} console, Terraform, API, or the {{site.data.keyword.cloud_notm}} CLI.
{: shortdesc}

## Key considerations before you begin
{: #key-considerations}

Review these requirements before you select a migration path. They apply to all interfaces and to both the read replica and backup and restore paths.

### Version requirement
{: #version-requirement}

PostgreSQL 18 is required on the Gen 1 source before either migration path can proceed. If your deployment is on an earlier version, you must complete an in-place upgrade on Gen 1 first. This upgrade is a separate service-impacting event, so plan it as a distinct maintenance activity before you schedule the migration.

The in-place upgrade supports direct version jumps, so you can upgrade from PostgreSQL 15, 16, or 17 directly to PostgreSQL 18 without intermediate steps. Backup and restore is also available as an alternative upgrade method.

| Source version on Gen 1 | Upgrade required | Upgrade method | Target version on Gen 2 |
| --- | --- | --- | --- |
| 15 | Yes | In-place upgrade (direct jump) or backup and restore | 18 |
| 16 | Yes | In-place upgrade (direct jump) or backup and restore | 18 |
| 17 | Yes | In-place upgrade (direct jump) or backup and restore | 18 |
| 18 | No | Not applicable | 18 |
{: caption="Version readiness for Gen 1 to Gen 2 migration" caption-side="bottom"}

For upgrade instructions, see [Upgrading PostgreSQL](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading-postgresql#upgrading-postgresql-in-place).

### Private endpoints
{: #private-endpoints-requirement}

Gen 2 deployments use private endpoints only. Public endpoints are not available on Gen 2. Before you start the migration, confirm that your application infrastructure can reach private service endpoints. If your applications currently connect over public endpoints, plan the required network changes before you begin.

For the read replica migration path, the Gen 1 source instance must also have a private service endpoint enabled before setting up replication, in addition to Gen 2 requiring private endpoints.

## Planning considerations
{: #planning-considerations}

Keep the following points in mind when you plan your migration:

* Deployments already on PostgreSQL 18 can complete the migration in a single event. Deployments on version 15, 16, or 17 require a separate upgrade on Gen 1 first. For more information, see [Version requirement](#version-requirement).
* For the read replica path, confirm that the Gen 1 source instance has a private service endpoint enabled before starting the migration.
* Gen 2 supports private endpoints only. Plan your VPE gateway and any VPN or VSI connectivity before you start the migration.
* For the backup and restore path, downtime duration depends on your data volume and the time needed to restore and validate the Gen 2 deployment.
* Test the full procedure in a non-production environment before scheduling a production cutover.

## Before you begin
{: #prereqs}

Confirm the following items before you start the migration:

* You have a Gen 1 {{site.data.keyword.databases-for-postgresql}} deployment to migrate.
* You have access to provision a Gen 2 deployment in the target account, resource group, and region.
* A maintenance window is approved for the migration activity.
* You have access to the application configuration so that you can update connection details during cutover.

Complete the following checks on the source deployment:

* Note the current PostgreSQL version and confirm it is on version 18, or plan the upgrade first.
* Confirm there are no in-progress operations such as scaling, backups, or upgrades.
* Verify that a recent successful backup exists.
* Check disk utilization and available capacity.
* Identify whether applications connect over public endpoints, private endpoints, or both.
* If using the read replica path, verify that the Gen 1 instance has a private service endpoint enabled.

Run the migration against a development or low-traffic instance first to validate the procedure before scheduling a production cutover.
{: tip}
  
## Migrating using the console
{: #console-migration}
{: ui}

The Migration hub guides you through the migration in a wizard, split into two phases. Phase 1 prepares and provisions your Gen 2 target. Phase 2 handles the data transfer or cutover and closes out the migration.

Choose the migration path that best fits your requirements:

* [Read replica migration](#read-replica-migration): minimal downtime, recommended for production workloads
* [Backup and restore migration](#backup-restore-migration): planned downtime, use when a maintenance window is available

## Read replica migration
{: #read-replica-migration}
{: ui}

The read replica path migrates your data through continuous replication, keeping the Gen 1 source running until you are ready to cut over. Downtime is limited to the final promotion event.

* [Read replica Phase 1: Prepare and provision](#rr-phase-1)
   * [Read replica Step 1: Select migration path](#rr-select-path)
   * [Read replica Step 2: Assess Gen 1](#rr-assess-gen1)
   * [Read replica Step 3: Create Gen 2 read replica](#rr-provision-replica)
* [Read replica Phase 2: Replicate and cut over](#rr-phase-2)
   * [Read replica Step 4: Set network access](#rr-set-network-access)
   * [Read replica Step 5: Replication](#rr-replication)
   * [Read replica Step 6: Cutover](#rr-cutover)
   * [Read replica Step 7: Finish migration](#rr-finish-migration)

## Read replica Phase 1: Prepare and provision
{: #rr-phase-1}
{: ui}

Phase 1 covers the preparation and provisioning steps required before replication can begin.

### Read replica Step 1: Select migration path
{: #rr-select-path}
{: step}
{: ui}

Open the Migration hub in the {{site.data.keyword.cloud_notm}} console. Under **Data services**, select **Eligible data instances** to see the Gen 1 Classic deployments in your account that are ready to migrate. Find your {{site.data.keyword.databases-for-postgresql}} instance and click **Open wizard**.

The wizard opens on the **Select migration path** screen. Three options are available:

| Path | Downtime | When to use |
| --- | --- | --- |
| Read replica | Minimal | Recommended for production workloads. Replication runs continuously until you choose to cut over. |
| Backup and restore | Planned | Use when a scheduled maintenance window is acceptable. See [Backup and restore migration](#backup-restore-migration). |
| Provision only | None (data not moved) | Use for development and test environments where data continuity is not required. This option opens the standard {{site.data.keyword.cloud_notm}} provisioning experience for {{site.data.keyword.databases-for-postgresql}} on Gen 2 with no data migration step. |
{: caption="Available migration paths" caption-side="bottom"}

Select **Read replica** and click **Start**.

### Read replica Step 2: Assess Gen 1
{: #rr-assess-gen1}
{: step}
{: ui}

After you select the read replica path, the wizard runs an automated assessment of your Gen 1 source deployment to confirm that it is ready to migrate. The assessment identifies any required actions and items that need your attention before you can proceed.

Before you proceed, confirm that the PostgreSQL 18 version requirement is met.

The wizard also provides recommendations and contextual information about what changes between Gen 1 and Gen 2. Review these items before you proceed, as some settings behave differently on Gen 2.

After you address any flagged items, click **Refresh** to update the assessment results. When the assessment is clear, click **Next: Create Gen 2** to continue.

### Read replica Step 3: Create Gen 2 read replica
{: #rr-provision-replica}
{: step}
{: ui}

This step provisions the Gen 2 (VPC) deployment as a read-only replica of your Gen 1 source. The Gen 1 source remains fully operational throughout. While replication is active, the Gen 2 instance is billed based on the configuration you choose.

The wizard shows a **Preview PostgreSQL on Gen 2** summary with the proposed configuration for the new deployment, covering details, resource allocations, service settings, and backup settings. Fields marked **Changed** highlight settings that differ from the Gen 1 source.

Click **Edit configurations** to adjust the Gen 2 settings before provisioning starts. The edit panel shows your Gen 1 resource allocation for reference and lets you configure:

* **Location and platform**: Target region. Platform is locked to Gen 2.
* **Service details**: Instance name, resource group, hosting model, and tags.
* **Resource allocation**: Host size (vCPU and RAM), disk and member count.
* **Service configuration**: Database version (18 preferred) and encryption.

Size the Gen 2 deployment to handle production traffic after promotion.

Disk allocation must match or exceed the Gen 1 source.
{: note}

After you confirm the configuration, the wizard starts provisioning and a progress bar tracks completion. You can open the resource list to monitor the new instance while the deployment runs. The **Next: Set network access** button becomes available after the Gen 2 instance is running.

## Read replica Phase 2: Replicate and cut over
{: #rr-phase-2}
{: ui}

Phase 2 covers the steps to configure network access, monitor replication, perform the cutover, and finish the migration.

### Read replica Step 4: Set network access
{: #rr-set-network-access}
{: step}
{: ui}

Configure private connectivity to the Gen 2 (VPC) instance before cutover. All connection types require a Virtual Private Endpoint (VPE) gateway for private access to the Gen 2 deployment.

Click **Open VPE gateway** to create the gateway from the VPC networking console. **Refresh** after setup to confirm the configuration is detected by the wizard.

The VPE gateway is required. The **Next: Replication** button is not available until this step is complete.
{: important}

### Read replica Step 5: Replication
{: #rr-replication}
{: step}
{: ui}

The Replication step shows the current synchronization status between your Gen 1 source and the Gen 2 read replica.

| Indicator | Description |
| --- | --- |
| Replication health status | **Active** means replication is running and healthy. |
| Replication lag | The time delay between the Gen 1 primary and the Gen 2 replica. |
| Started | When replication began. |
{: caption="Replication health indicators" caption-side="bottom"}

Do not proceed to cutover until replication health shows **Active** and the lag is within an acceptable threshold for your workload.
{: important}

Use this time to confirm that the Gen 2 deployment is reachable through the VPE private endpoint, inform application owners of the connection details they need to update, and document your rollback plan before you initiate promotion.

### Read replica Step 6: Cutover
{: #rr-cutover}
{: step}
{: ui}

The cutover step handles the final switchover. The wizard breaks this into two stages: stopping writes and promoting the replica, then updating application endpoints.

#### Stage 1: Stop writes and promote
{: #rr-stop-writes-promote}
{: ui}

**Stop writes to Gen 1**

Set the Gen 1 source to read-only before promoting the replica. This prevents any new data from being written to Gen 1 after the replica is promoted. You can also disconnect all active connections to Gen 1 to ensure a clean cutoff before promotion.

Stop writes only after you confirm that replication is complete. Stopping writes while the lag is still high can cause data loss or inconsistency.
{: important}

**Promote the read replica**

Promoting the replica makes the Gen 2 deployment the new primary. When you promote:

* Replication stops.
* The replica becomes an independent instance.
* Backups are enabled on the new Gen 2 (VPC) primary instance.

Promotion cannot be undone. Confirm that replication is complete and your rollback plan is in place before you click **Promote read replica**.
{: important}

#### Stage 2: Update applications
{: #rr-cutover-update-apps}
{: ui}

After the replica is promoted, the wizard provides the Gen 2 connection details and guides you through updating your applications.

**Update application endpoints**

Update your application to use the Gen 2 (VPC) endpoint instead of the Gen 1 (Classic) endpoint. The wizard provides the full connection endpoint string, hostname, port, database name, and SSL mode for easy reference.

Gen 2 uses Let's Encrypt certificates rather than self-signed TLS certificates. Update any certificate validation or pinning in your application accordingly.
{: note}

**Update authentication credentials**

If your application already uses Gen 2 service credentials, no changes are needed. Otherwise, create a new service credential for the Gen 2 deployment and update your application configuration.

**Redeploy or restart applications**

Redeploy or restart your applications to apply the updated endpoint and credentials. Confirm that they can read existing data and write new data on Gen 2 before proceeding.

### Read replica Step 7: Finish migration
{: #rr-finish-migration}
{: step}
{: ui}

After you confirm that your applications are running on Gen 2, click **Next: Finish migration**.

The Finish migration screen shows a side-by-side summary of both instances, including instance status, active connections, last backup timestamp, and disk usage. Additional metrics for both instances are available through IBM Cloud Monitoring.

After finishing, the Gen 1 deployment stays available in a read-only state. Keep it running through your planned retention period before decommissioning, in line with your rollback policy.

Continue to [Post-migration tasks](#post-migration) to wrap up.

## Backup and restore migration
{: #backup-restore-migration}
{: ui}

The backup and restore path provisions a new Gen 2 deployment from a backup of the Gen 1 source. For a production migration, stopping writes to Gen 1 before taking the backup ensures that the restore captures the latest data. For dev or test migrations, you can skip the write-stop step and restore from the most recent automatic backup instead.

* [Backup and restore Phase 1: Prepare and provision](#br-phase-1)
   * [Backup and restore Step 1: Select migration path](#br-select-path)
   * [Backup and restore Step 2: Assess Gen 1](#br-assess-gen1)
   * [Backup and restore Step 3: Backup Gen 1](#br-backup-gen1)
   * [Backup and restore Step 4: Create Gen 2 from backup](#br-provision-gen2)
* [Backup and restore Phase 2: Connect and cut over](#br-phase-2)
   * [Backup and restore Step 5: Set network access](#br-set-network-access)
   * [Backup and restore Step 6: Cutover](#br-cutover)
   * [Backup and restore Step 7: Finish migration](#br-finish-migration)

## Backup and restore Phase 1: Prepare and provision
{: #br-phase-1}
{: ui}

Phase 1 covers the preparation, backup, and provisioning steps required before the Gen 2 instance is ready.

### Backup and restore Step 1: Select migration path
{: #br-select-path}
{: step}
{: ui}

Open the Migration hub in the {{site.data.keyword.cloud_notm}} console. Under **Data services**, select **Eligible data instances** to see the Gen 1 Classic deployments in your account that are ready to migrate. Find your {{site.data.keyword.databases-for-postgresql}} instance, click **Open wizard**, then select **Backup and restore** and click **Start**.

### Backup and restore Step 2: Assess Gen 1
{: #br-assess-gen1}
{: step}
{: ui}

After you select the backup and restore path, the wizard runs an automated assessment of your Gen 1 source deployment to confirm that it is ready to migrate. The assessment identifies any required actions and items that need your attention before you can proceed.

Before you proceed, confirm that the PostgreSQL 18 version requirement is met.

The wizard also provides recommendations and contextual information about what changes between Gen 1 and Gen 2. Review these items before you proceed, as some settings behave differently on Gen 2.

After you address any flagged items, click **Refresh** to update the assessment results. When the assessment is clear, click **Next: Backup Gen 1** to continue.

### Backup and restore Step 3: Backup Gen 1
{: #br-backup-gen1}
{: step}
{: ui}

This step takes a backup of the Gen 1 source to use as the restore source for the new Gen 2 deployment. The step has two parts: optionally stopping writes, then selecting a backup option.

#### Stop writes to Gen 1 (optional for production, skip for dev/test)
{: #br-stop-writes}
{: ui}

For a production migration, set the Gen 1 (Classic) source to read-only before creating the backup. This ensures that the backup captures a consistent final state with no in-flight writes. You can also disconnect all active connections to prevent any new writes during the backup window.

Do not stop writes for dev or test migrations. Setting the source to read-only before you are ready for final cutover can result in data loss on the active instance.
{: important}

After writes are stopped, the wizard shows a **Re-enable writes to Gen 1** option in case you need to roll back before taking the backup.

#### Select a backup option
{: #br-select-backup}
{: ui}

The wizard presents two backup options:

| Option | Description | Best for |
| --- | --- | --- |
| Create on-demand backup | Creates a new backup so the restore uses the latest available data from the Gen 1 source. | Production migrations where data completeness is required. |
| Use last backup | Restores from the most recent completed automatic backup. Changes made after that backup are not included. | Dev and test migrations where speed matters more than data completeness. |
{: caption="Backup options for Gen 1" caption-side="bottom"}

For a production migration, use **Create on-demand backup** to ensure that the restore includes the latest data.
{: tip}

### Backup and restore Step 4: Create Gen 2 from backup
{: #br-provision-gen2}
{: step}
{: ui}

This step provisions a new Gen 2 (VPC) instance restored from the Gen 1 backup. The Gen 1 source remains available and continues to be billed while the new instance is provisioned.

The wizard shows a **Preview PostgreSQL on Gen 2** summary with the proposed configuration, covering details, resource allocations, service settings, and backup settings. Fields marked **Changed** highlight differences from the Gen 1 source.

Click **Edit configurations** to adjust the Gen 2 settings before provisioning begins. The edit panel shows your Gen 1 resource allocation for reference and lets you configure:

* **Location and platform**: Target region. Platform is locked to Gen 2.
* **Service details**: Instance name, resource group, hosting model, and tags.
* **Resource allocation**: Host size (vCPU and RAM), disk and member count.
* **Service configuration**: Database version (18 preferred) and encryption.

Size the Gen 2 deployment to handle production traffic.

Disk allocation must match or exceed the Gen 1 source.
{: note}

After you confirm, the wizard starts the restore and a progress bar tracks completion. The **Next: Set network access** button becomes available after the Gen 2 instance is running.

## Backup and restore Phase 2: Connect and cut over
{: #br-phase-2}
{: ui}

Phase 2 covers the steps to configure network access, perform the cutover, and finish the migration.

### Backup and restore Step 5: Set network access
{: #br-set-network-access}
{: step}
{: ui}

Configure private connectivity to the Gen 2 (VPC) instance before cutover. All connection types require a Virtual Private Endpoint (VPE) gateway for private access to the Gen 2 deployment.

Click **Open VPE gateway** to create the gateway from the VPC networking console. **Refresh** after setup to confirm the configuration is detected by the wizard.

The VPE gateway is required. The **Next: Cutover** button is not available until this step is complete.
{: important}

### Backup and restore Step 6: Cutover
{: #br-cutover}
{: step}
{: ui}

With the backup and restore path, the Gen 1 source is in read-only state if you stopped writes in Phase 1. The cutover step focuses on switching your applications to the new Gen 2 deployment.

**Update application endpoints**

Update your application to use the Gen 2 (VPC) endpoint instead of the Gen 1 (Classic) endpoint. The wizard provides the full connection endpoint string, hostname, port, database name, and SSL mode for easy reference.

Gen 2 uses Let's Encrypt certificates rather than self-signed TLS certificates. Update any certificate validation or pinning in your application accordingly.
{: note}

**Update authentication credentials**

If your application already uses Gen 2 service credentials, no changes are needed. Otherwise, create a new service credential for the Gen 2 deployment and update your application configuration.

**Redeploy or restart applications**

Redeploy or restart your applications to apply the updated endpoint and credentials. Confirm that they can read existing data and write new data on Gen 2 before proceeding.

### Backup and restore Step 7: Finish migration
{: #br-finish-migration}
{: step}
{: ui}

After you confirm that your applications are running on Gen 2, click **Next: Finish migration** to complete the workflow.

The Finish migration screen shows a side-by-side summary of both instances, including instance status, active connections, last backup timestamp, and disk usage. Additional metrics for both instances are available through IBM Cloud Monitoring.

After finishing, the Gen 1 deployment stays available. Keep it running through your planned retention period before decommissioning, in line with your rollback policy.

Continue to [Post-migration tasks](#post-migration) to wrap up.

---

## Migrating using the CLI
{: #cli-migration}
{: cli}

You can migrate your {{site.data.keyword.databases-for-postgresql}} instance from Gen 1 (Classic) to Gen 2 (VPC) by using the {{site.data.keyword.cloud_notm}} CLI. Choose between two paths:

* [Read replica migration with the CLI](#cli-read-replica-migration): Minimal downtime, recommended for production workloads.
* [Backup and restore migration with the CLI](#cli-backup-restore-migration): Planned downtime, use when a maintenance window is available.

Before you begin:

- Install the [{{site.data.keyword.cloud_notm}} CLI](/docs/cli?topic=cli-getting-started){: external}.
- Install the  {{site.data.keyword.databases-for}} plug-in:

   ```bash
   ibmcloud plugin install cloud-databases -f
   ```
   {: pre}

- Log in and target the resource group where the Gen 2 instance will be created:

   ```bash
   ibmcloud login --apikey "$IBMCLOUD_API_KEY"
   ibmcloud target -g <resource-group>
   ```
   {: pre}

Migration to Gen 2 is initially available in the us-east (Washington, DC) region, with additional regions to follow. Ensure that the region you specify is a supported Gen 2 region, such as `us-east`.
{: note}

### Read replica migration with the CLI
{: #cli-read-replica-migration}
{: cli}

The read replica path migrates your data through continuous replication, keeping the Gen 1 source running until you are ready to cut over. Downtime is limited to the final promotion event. When replication is caught up and writes to Gen 1 are stopped, you promote the replica to an independent primary Gen 2 instance.

#### Phase 1: Verify prerequisites and provision Gen 2 replica
{: #cli-rr-phase-1}
{: cli}

1. **Set environment variables.**

   ```bash
   export CLASSIC_NAME=<gen1-instance-name>
   export GEN2_NAME=<gen2-instance-name>
   export GEN2_REGION=us-east
   ```
   {: pre}

2. **Get the Gen 1 CRN.**

   The source CRN must be the Gen 1 **instance** CRN (ending with `::`), not a backup CRN.

   ```bash
   CLASSIC_CRN="$(ibmcloud resource service-instance "$CLASSIC_NAME" --output json | jq -r '.[0].crn')"
   echo "$CLASSIC_CRN"
   ```
   {: pre}

3. **Verify the PostgreSQL version on Gen 1.**

   Confirm that the source deployment is running PostgreSQL 18:

   ```bash
   ibmcloud cdb deployment-about "$CLASSIC_CRN"
   ```
   {: pre}

   If the source is not on version 18, complete an in-place upgrade on Gen 1 first. See [Version requirement](#version-requirement).

4. **Verify the Gen 1 private endpoint is enabled.**

   Verify that the Gen 1 source instance has a private service endpoint enabled:

   ```bash
   ibmcloud resource service-instance "$CLASSIC_NAME" --output json | jq '.[0].parameters.service_endpoints'
   ```
   {: pre}

   If the Gen 1 instance only has a public endpoint, enable private endpoints (or both public and private):

   ```bash
   ibmcloud resource service-instance-update "$CLASSIC_NAME" --service-endpoints private
   ```
   {: pre}

   Replication cannot be set up unless the Gen 1 source has a private endpoint enabled.
   {: important}

5. **Prepare the provisioning parameters file.**

   Create a `params.json` file with the following content. The `storage_gb` value is per member, in GB, and must meet or exceed the Gen 1 source data volume:

   ```json
   {
     "dataservices": {
       "postgresql": {
         "version": "18",
         "members": 2,
         "storage_gb": 10,
         "host_flavor": "bx3d.4x20",
         "read_replica": {
           "source_crn": "<full Gen 1 instance CRN>",
           "source_type": "gen1"
         }
       }
     }
   }
   ```
   {: codeblock}

   Or generate it using `jq` from the `$CLASSIC_CRN` captured earlier:

   ```bash
   jq --arg crn "$CLASSIC_CRN" '.dataservices.postgresql.read_replica.source_crn = $crn' params.json.example > params.json
   ```
   {: pre}

   Pass the Gen 1 **instance** CRN in `dataservices.postgresql.read_replica.source_crn`. Do not pass a backup CRN and do not include `restore_backup_id`. That parameter is for the backup and restore path only.
   {: important}

6. **Create the Gen 2 read replica.**

   ```bash
   ibmcloud resource service-instance-create "$GEN2_NAME" \
     databases-for-postgresql standard-gen2 "$GEN2_REGION" \
     -p @params.json
   ```
   {: pre}

7. **Wait until the instance is active.**

   Provisioning and initial data replication can take several hours depending on data volume. Poll the status until `State` shows `active`:

   ```bash
   ibmcloud resource service-instance "$GEN2_NAME"
   ```
   {: pre}

   Do not delete the Gen 1 instance while replication is in progress. The Gen 1 source must remain available until after you promote the replica and complete cutover.
   {: important}

#### Phase 2: Replicate, cut over, and finish
{: #cli-rr-phase-2}
{: cli}

1. **Configure a VPE gateway for private network access.**

   Gen 2 supports private endpoints only. Before switching applications, create a Virtual Private Endpoint (VPE) gateway from the [{{site.data.keyword.cloud_notm}} VPC networking console](https://cloud.ibm.com/infrastructure/provision/endpointGateway){: external} to connect to the new Gen 2 instance.

2. **Verify replication health and lag.**

   Connect to the Gen 1 leader instance with `psql` and check the replication status:

   ```sql
   SELECT * FROM pg_stat_replication;
   ```
   {: codeblock}

   Confirm that replication is active and lag is minimal before initiating cutover.

3. **Stop writes to Gen 1.**

   Put your applications in maintenance mode or stop writes to the Gen 1 source instance to ensure all remaining transactions are replicated before promotion. Do not proceed while the replication lag is still high.
   {: important}

4. **Promote the Gen 2 read replica.**

   Promote the Gen 2 replica to an independent standalone deployment:

   ```bash
   ibmcloud cdb read-replica-promote "$GEN2_NAME"
   ```
   {: pre}

   After promotion, replication stops, the instance becomes an independent primary database that accepts reads and writes, and automated Gen 2 backups are enabled. Promotion cannot be undone.
   {: note}

5. **Create service credentials.**

   ```bash
   ibmcloud resource service-key-create "${GEN2_NAME}-credentials" \
     --instance-name "$GEN2_NAME" \
     -p '{"role_crn": "crn:v1:bluemix:public:iam::::serviceRole:Manager"}'
   ```
   {: pre}

   Retrieve the credentials and connection details:

   ```bash
   ibmcloud resource service-key "${GEN2_NAME}-credentials" --output json
   ```
   {: pre}

   Pass `role_crn` as a JSON parameter (`-p '{"role_crn": "crn:v1:bluemix:public:iam::::serviceRole:Manager"}'`), not as the positional `ROLE_NAME` argument. The `databases-for-postgresql` service does not have a `Manager` IAM role, and the CLI fails if `Manager` is passed positionally.
   {: note}

6. **Update application endpoints.**

   Update your application to use the Gen 2 private endpoint from the credentials output instead of the Gen 1 endpoint.

   Gen 2 uses Let's Encrypt certificates rather than self-signed TLS certificates. Update any certificate validation or pinning in your application accordingly.

7. **Redeploy or restart applications.**

   Restart your applications with the updated endpoint and credentials. Confirm that they can read existing data and write new data on Gen 2 before proceeding.

8. **Finish and decommission Gen 1.**

   After validating that your applications are running correctly on Gen 2, proceed to [Post-migration tasks](#post-migration). Keep the Gen 1 deployment available through your planned retention period before decommissioning it:

   ```bash
   # ibmcloud resource service-instance-delete "$CLASSIC_NAME"   # only after cutover and retention period
   ```
   {: pre}

### Backup and restore migration with the CLI
{: #cli-backup-restore-migration}
{: cli}

The backup and restore path creates a new Gen 2 instance from a backup of the Gen 1 source. All writes to Gen 1 after the backup are not included in the Gen 2 instance. 
Plan a maintenance window and stop application writes before taking a backup to ensure a consistent migration.

#### Phase 1: Capture backup and provision Gen 2
{: #cli-br-phase-1}
{: cli}

1. **Set environment variables.**

   ```bash
   export CLASSIC_NAME=<gen1-instance-name>
   export GEN2_NAME=<gen2-instance-name>
   export GEN2_REGION=us-east
   ```
   {: pre}

2. **Get the Gen 1 CRN.**

   ```bash
   CLASSIC_CRN="$(ibmcloud resource service-instance "$CLASSIC_NAME" --output json | jq -r '.[0].crn')"
   echo "$CLASSIC_CRN"
   ```
   {: pre}

3. **Verify the PostgreSQL version on Gen 1.**

   Confirm that the source deployment is running PostgreSQL 18:

   ```bash
   ibmcloud cdb deployment-about "$CLASSIC_CRN"
   ```
   {: pre}

   If the source is not on version 18, complete an in-place upgrade on Gen 1 first. See [Version requirement](#version-requirement).

4. **Stop writes to Gen 1 and take an on-demand backup.**

   For production migrations, stop application writes to ensure a consistent restore point, then trigger a backup. For development or test migrations where data completeness is not required, you can skip this step.

   ```bash
   ibmcloud cdb backup-now "$CLASSIC_CRN"
   ```
   {: pre}

5. **Retrieve the backup CRN.**

   List backups and wait until one is completed and restorable:

   ```bash
   ibmcloud cdb backups "$CLASSIC_CRN"
   ```
   {: pre}

   The `restore_backup_id` must be the full coupled backup CRN in the following format:

   ```text
   crn:v1:bluemix:public:databases-for-postgresql:<region>:a/<account_id>:<instance_id>:backup:<backup_id>
   ```
   {: screen}

   If the output shows only a backup UUID, compose the full CRN from the Gen 1 CRN:

   ```bash
   BACKUP_UUID=<backup-guid>
   BACKUP_CRN="${CLASSIC_CRN%::}:backup:${BACKUP_UUID}"
   ```
   {: pre}

   Passing only the backup UUID causes provisioning to fail. You must use the full backup CRN.
   {: important}

6. **Prepare the provisioning parameters file.**

   Create a `params.json` file with the following content. The `storage_gb` value is per member, in GB, and must meet or exceed the Gen 1 source data volume:

   ```json
   {
     "dataservices": {
       "postgresql": {
         "version": "18",
         "members": 2,
         "storage_gb": 10,
         "host_flavor": "bx3d.4x20"
       },
       "restore_backup_id": "<full backup CRN from the previous step>"
     }
   }
   ```
   {: codeblock}

   Or generate it from the variable captured earlier:

   ```bash
   jq --arg crn "$BACKUP_CRN" '.dataservices.restore_backup_id = $crn' params.json.example > params.json
   ```
   {: pre}

   Do not use the flat `{"backup_id": ...}` format that is used for Classic restores. In Gen 2, specify the backup CRN in `dataservices.restore_backup_id`.

7. **Create the Gen 2 instance from the backup.**

   ```bash
   ibmcloud resource service-instance-create "$GEN2_NAME" \
     databases-for-postgresql standard-gen2 "$GEN2_REGION" \
     -p @params.json
   ```
   {: pre}

8. **Wait until the instance is active.**

   Restore can take several hours depending on data volume. Poll the status until `State` shows `active`:

   ```bash
   ibmcloud resource service-instance "$GEN2_NAME"
   ```
   {: pre}

   Do not delete the Gen 1 instance while the restore is running. Deleting the source also deletes its backups.
   {: important}

#### Phase 2: Configure access, cut over, and finish
{: #cli-br-phase-2}
{: cli}

1. **Configure a VPE gateway for private network access.**

   Gen 2 supports private endpoints only. Before switching applications, create a Virtual Private Endpoint (VPE) gateway from the [{{site.data.keyword.cloud_notm}} VPC networking console](https://cloud.ibm.com/infrastructure/provision/endpointGateway){: external} to connect to the new Gen 2 instance.

2. **Create service credentials.**

   ```bash
   ibmcloud resource service-key-create "${GEN2_NAME}-credentials" \
     --instance-name "$GEN2_NAME" \
     -p '{"role_crn": "crn:v1:bluemix:public:iam::::serviceRole:Manager"}'
   ```
   {: pre}

   Retrieve the credentials and connection details:

   ```bash
   ibmcloud resource service-key "${GEN2_NAME}-credentials" --output json
   ```
   {: pre}

   Pass `role_crn` as a JSON parameter (`-p '{"role_crn": "crn:v1:bluemix:public:iam::::serviceRole:Manager"}'`), not as the positional `ROLE_NAME` argument. The `databases-for-postgresql` service does not have a `Manager` IAM role, and the CLI fails if `Manager` is passed positionally.
   {: note}

3. **Update application endpoints.**

   Update your application to use the Gen 2 private endpoint from the credentials output instead of the Gen 1 endpoint.

   Gen 2 uses Let's Encrypt certificates rather than self-signed TLS certificates. Update any certificate validation or pinning in your application accordingly.

4. **Redeploy or restart applications.**

   Restart your applications with the updated endpoint and credentials. Confirm that they can read existing data and write new data on Gen 2 before proceeding.

5. **Finish and decommission Gen 1.**

   After validating that your applications are running correctly on Gen 2, proceed to [Post-migration tasks](#post-migration). Keep the Gen 1 deployment available through your planned retention period before decommissioning it:

   ```bash
   # ibmcloud resource service-instance-delete "$CLASSIC_NAME"   # only after cutover and retention period
   ```
   {: pre}

---

## Migrating using Terraform
{: #migrating-using-terraform}
{: terraform}

You can migrate your {{site.data.keyword.databases-for-postgresql}} instances from Gen 1 (Classic) to Gen 2 (VPC) by using Terraform and the {{site.data.keyword.cloud_notm}} provider (`ibm_database` resource).

Like the console wizard, Terraform migrations support both the **Read replica** (minimal downtime) and **Backup and restore** (planned downtime) paths.

Before you begin:

- [Install the Terraform CLI and the IBM Cloud Provider plug-in](/docs/ibm-cloud-provider-for-terraform?topic=ibm-cloud-provider-for-terraform-setup_cli#tf_installation){: external}.
- Ensure that you have an [IBM Cloud API key](/docs/account?topic=account-userapikey&interface=ui#create_user_key){: external}.

Gen 2 is initially available in the us-east (Washington, DC) region, with additional regions to follow. Ensure that the location parameter in your configurations specifies a supported Gen 2 region, such as us-east.
{: note}

Jump to the path you need:

* [Read replica migration with Terraform](#tf-read-replica-migration)
* [Backup and restore migration with Terraform](#tf-backup-restore-migration)
* [Configuring network access and credentials with Terraform](#tf-network-and-credentials)

### Read replica migration with Terraform
{: #tf-read-replica-migration}
{: terraform}

The read replica path uses asynchronous replication from the Gen 1 source to a new Gen 2 replica deployment. When replication is caught up and writes to Gen 1 are stopped, you promote the replica to a standalone Gen 2 primary instance by clearing `remote_leader_id`.

![Replication flow diagram — Gen 1 to Gen 2 read replica migration, promotion, and cutover](images/migrate-replica-flow.svg)
{: #tf-rr-migration-diagram}
{: caption="Read replica migration data flow" caption-side="bottom"}

#### Phase 1: Prepare and provision Gen 2 replica
{: #tf-rr-phase-1}
{: terraform}

1. **Verify and upgrade Gen 1 to PostgreSQL 18**

   Ensure that your Gen 1 source resource definition is updated to PostgreSQL version 18 before creating the Gen 2 replica:

   ```hcl
   resource "ibm_database" "gen1_postgres" {
     name              = "pg-gen1-production"
     service           = "databases-for-postgresql"
     plan              = "standard"
     location          = "us-east"
     resource_group_id = var.resource_group_id
     version           = "18"
     adminpassword     = var.admin_password
   }
   ```
   {: codeblock}

   Apply the upgrade:
   ```bash
   terraform apply -target=ibm_database.gen1_postgres
   ```
   {: pre}

   The Gen 1 source instance must have a private service endpoint enabled before creating the Gen 2 read replica. If your Gen 1 instance has only public endpoints enabled, configure `service_endpoints = "private"` (or `"public-and-private"`) on the Gen 1 resource and apply the change.
   {: important}

2. **Provision Gen 2 read replica linked to Gen 1**

   Define the Gen 2 instance with `plan = "standard-gen2"`, `service_endpoints = "private"`, and `remote_leader_id` pointing to the Gen 1 instance ID:

   ```hcl
   resource "ibm_database" "gen2_postgres" {
     name              = "pg-gen2-production"
     service           = "databases-for-postgresql"
     plan              = "standard-gen2"
     location          = "us-east"
     resource_group_id = var.resource_group_id
     version           = "18"

     # Link to Gen 1 Classic source instance
     remote_leader_id  = ibm_database.gen1_postgres.id

     # Gen 2 only supports private service endpoints
     service_endpoints = "private"

     group {
       group_id = "member"

       host_flavor {
         id = "bx3d.4x20"
       }

       disk {
         allocation_mb = 20480
       }
     }

     tags = ["env:production", "migration:gen1-to-gen2"]
   }
   ```
   {: codeblock}

   Apply the configuration:
   ```bash
   terraform apply -target=ibm_database.gen2_postgres
   ```
   {: pre}

3. **Configure private network access**

   Gen 2 instances only support private endpoints. Before proceeding to cutover, set up a Virtual Private Endpoint (VPE) gateway as described in [Configuring network access and credentials with Terraform](#tf-network-and-credentials).

#### Phase 2: Cutover, promote replica, and switch applications
{: #tf-rr-phase-2}
{: terraform}

1. **Verify replication status.**

   Confirm that replication health is active and lag is at an acceptable level before proceeding.

2. **Stop application writes to Gen 1.**

   Put your applications in maintenance mode or stop writes to ensure a consistent cutoff point.

3. **Promote Gen 2 replica to primary.**

   In your Terraform configuration, promote the replica by clearing `remote_leader_id` (set to `""` or remove the attribute):

   ```hcl
   resource "ibm_database" "gen2_postgres" {
     name              = "pg-gen2-production"
     service           = "databases-for-postgresql"
     plan              = "standard-gen2"
     location          = "us-east"
     resource_group_id = var.resource_group_id
     version           = "18"

     # Clear remote_leader_id to promote to standalone primary instance
     remote_leader_id  = ""

     service_endpoints = "private"

     group {
       group_id = "member"

       host_flavor {
         id = "bx3d.4x20"
       }

       disk {
         allocation_mb = 20480
       }
     }

     tags = ["env:production", "migrated:gen2"]
   }
   ```
   {: codeblock}

   Apply the promotion:
   ```bash
   terraform apply -target=ibm_database.gen2_postgres
   ```
   {: pre}

   After promotion, replication stops, the instance becomes an independent primary database, and automated Gen 2 backups are enabled.
   {: note}

4. **Generate credentials and switch applications.**

   Create credentials for the Gen 2 instance and update application connection endpoints:

   ```hcl
   resource "ibm_resource_key" "gen2_service_credentials" {
     name                 = "pg-gen2-app-credentials"
     resource_instance_id = ibm_database.gen2_postgres.id
     parameters = {
       role_crn = "crn:v1:bluemix:public:iam::::serviceRole:Manager"
     }
   }

   output "gen2_connection_strings" {
     value     = ibm_resource_key.gen2_service_credentials.credentials
     sensitive = true
   }
   ```
   {: codeblock}

   Restart application workloads and validate database connectivity.

   Generate the new Gen 2 service credentials as described in [Configuring network access and credentials with Terraform](#tf-network-and-credentials), update your application connection settings with the Gen 2 private endpoints and Let's Encrypt certificates, and restart your workloads.

### Backup and restore migration with Terraform
{: #tf-backup-restore-migration}
{: terraform}

The backup and restore path restores a Gen 1 backup onto a newly provisioned Gen 2 instance.

#### Phase 1: Capture backup and restore to Gen 2
{: #tf-br-phase-1}
{: terraform}

1. **Verify PostgreSQL 18 on Gen 1.**

   Confirm that the source deployment is running PostgreSQL 18 before proceeding.

2. **Stop writes and capture an on-demand backup.**

   Stop application writes, then trigger an on-demand backup of the Gen 1 instance:

   ```bash
   ibmcloud cdb deployment-backup-now <gen1-instance-name-or-crn>
   ```
   {: pre}

   Retrieve the backup CRN:

   ```bash
   ibmcloud cdb deployment-backups <gen1-instance-name-or-crn>
   ```
   {: pre}

3. **Provision Gen 2 instance from backup.**

   Specify `backup_id` with the **full backup CRN** in your Gen 2 resource definition.

   The `backup_id` parameter requires the full CRN format of the backup:

   `crn:v1:bluemix:public:databases-for-postgresql:<region>:a/<account_id>:<instance_id>:backup:<backup_id>`

   Example:

   `crn:v1:bluemix:public:databases-for-postgresql:us-east:a/df5f422898b540846cefeb9c5ad0aa66:623ebc19-ad61-47f2-84c5-b5d5b8c7a15a:backup:8d29c132-f351-4067-9b7b-0f307fe0ef84`

   Passing only the backup UUID will cause provisioning to fail.
   {: important}

   ```hcl
   resource "ibm_database" "gen2_from_backup" {
     name              = "pg-gen2-restored"
     service           = "databases-for-postgresql"
     plan              = "standard-gen2"
     location          = "us-east"
     resource_group_id = var.resource_group_id
     version           = "18"

     # Restore from Gen 1 backup CRN (must be the full CRN, not just the UUID)
     backup_id         = var.gen1_backup_crn

     service_endpoints = "private"

     group {
       group_id = "member"

       host_flavor {
         id = "bx3d.4x20"
       }

       disk {
         allocation_mb = 20480
       }
     }

     tags = ["env:production", "restored-from:gen1"]
   }
   ```
   {: codeblock}

   Apply the restore:

   ```bash
   terraform apply -target=ibm_database.gen2_from_backup
   ```
   {: pre}

#### Phase 2: Network access and application cutover
{: #tf-br-phase-2}
{: terraform}

1. **Configure private network access and credentials.**

   Follow the shared steps in [Configuring network access and credentials with Terraform](#tf-network-and-credentials) to configure the VPE gateway for `ibm_database.gen2_from_backup.crn` and generate new service credentials.

2. **Switch application endpoints.**

   Update application connection strings with the new private endpoints and Let's Encrypt certificates, then restart your applications.

3. **Generate credentials and switch applications.**

   Create credentials for the Gen 2 instance and update application connection endpoints:

   ```hcl
   resource "ibm_resource_key" "gen2_service_credentials" {
     name                 = "pg-gen2-app-credentials"
     resource_instance_id = ibm_database.gen2_from_backup.id
     parameters = {
       role_crn = "crn:v1:bluemix:public:iam::::serviceRole:Manager"
     }
   }

   output "gen2_connection_strings" {
     value     = ibm_resource_key.gen2_service_credentials.credentials
     sensitive = true
   }
   ```
   {: codeblock}

   Restart application workloads and validate database connectivity.

### Configuring network access and credentials with Terraform
{: #tf-network-and-credentials}
{: terraform}

Because Gen 2 deployments only support private service endpoints and use a different credential model than Gen 1, all Gen 2 deployments require a Virtual Private Endpoint (VPE) gateway and new service credentials regardless of the migration path used.

#### 1. Configure VPE gateway for private network access
{: #tf-vpe-setup}
{: terraform}

Create a Virtual Private Endpoint (VPE) gateway in your VPC for your Gen 2 database instance CRN:

```hcl
resource "ibm_is_virtual_endpoint_gateway" "pg_vpe" {
  name           = "vpe-pg-gen2"
  target {
    crn           = ibm_database.gen2_postgres.crn # Or ibm_database.gen2_from_backup.crn
    resource_type = "provider_cloud_service"
  }
  vpc            = var.vpc_id
  resource_group = var.resource_group_id
}
```
{: codeblock}

Apply the VPE gateway:

```bash
terraform apply -target=ibm_is_virtual_endpoint_gateway.pg_vpe
```
{: pre}

#### 2. Generate Gen 2 service credentials
{: #tf-credentials-setup}
{: terraform}

Create service credentials for the Gen 2 instance and export the connection strings:

```hcl
resource "ibm_resource_key" "gen2_service_credentials" {
  name                 = "pg-gen2-app-credentials"
  resource_instance_id = ibm_database.gen2_postgres.id # Or ibm_database.gen2_from_backup.id
  parameters = {
    role_crn = "crn:v1:bluemix:public:iam::::serviceRole:Manager"
  }
}

output "gen2_connection_strings" {
  value     = ibm_resource_key.gen2_service_credentials.credentials
  sensitive = true
}
```
{: codeblock}

Apply the credentials:

```bash
terraform apply -target=ibm_resource_key.gen2_service_credentials
```
{: pre}

---

## Migrating using the API
{: #api-migration}
{: api}

You can migrate your {{site.data.keyword.databases-for-postgresql}} instance from Gen 1 (Classic) to Gen 2 (VPC) by using the [{{site.data.keyword.cloud_notm}} Resource Controller API](https://cloud.ibm.com/apidocs/resource-controller/resource-controller){: external} and the [{{site.data.keyword.databases-for}} API](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-api){: external}. Choose between two paths:

* [Read replica migration with the API](#api-read-replica-migration): Minimal downtime, recommended for production workloads.
* [Backup and restore migration with the API](#api-backup-restore-migration): Planned downtime, use when a maintenance window is available.

Before you begin, obtain an IAM token:

```bash
IAM_TOKEN=$(curl -s -X POST "https://iam.cloud.ibm.com/identity/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=urn:ibm:params:oauth:grant-type:apikey&apikey=$IBMCLOUD_API_KEY" \
  | jq -r '.access_token')
```
{: pre}

Migration to Gen 2 is initially available in the `us-east` (Washington, DC) region, with additional regions to follow. Ensure that the region you specify is a supported Gen 2 region, such as `us-east`.
{: note}

### Read replica migration with the API
{: #api-read-replica-migration}
{: api}

The read replica path migrates your data through continuous replication, keeping the Gen 1 source running until you are ready to cut over. Downtime is limited to the final promotion event. When replication is caught up and writes to Gen 1 are stopped, you promote the replica to an independent primary Gen 2 instance using the API.

#### Phase 1: Verify prerequisites and provision Gen 2 replica
{: #api-rr-phase-1}
{: api}

1. **Get the Gen 1 resource group ID, CRN, and GUID.**

   Retrieve the resource group ID for the resource group where you plan to create the Gen 2 instance:

   ```bash
   curl -s -X GET \
     "https://resource-controller.cloud.ibm.com/v2/resource_groups?account_id=$ACCOUNT_ID" \
     -H "Authorization: Bearer $IAM_TOKEN" | jq '.resources[] | {name, id}'
   ```
   {: pre}

   Retrieve the Gen 1 instance CRN and GUID. The source CRN must be the Gen 1 **instance** CRN (ending with `::`), not a backup CRN:

   ```bash
   CLASSIC_RESPONSE=$(curl -s -X GET \
     "https://resource-controller.cloud.ibm.com/v2/resource_instances?name=$CLASSIC_NAME" \
     -H "Authorization: Bearer $IAM_TOKEN")
   CLASSIC_CRN=$(echo "$CLASSIC_RESPONSE" | jq -r '.resources[0].crn')
   CLASSIC_GUID=$(echo "$CLASSIC_RESPONSE" | jq -r '.resources[0].guid')
   echo "CRN: $CLASSIC_CRN"
   echo "GUID: $CLASSIC_GUID"
   ```
   {: pre}

2. **Verify the PostgreSQL version on Gen 1.**

   Confirm that the source deployment is running PostgreSQL version 18 before proceeding. If the source is not on version 18, complete an in-place upgrade on Gen 1 first. For more information, see [Version requirement](#version-requirement).

   Set `GEN2_REGION` to the target Gen 2 region (for example, `us-east`):

   ```bash
   GEN2_REGION=<target-gen2-region>
   ```
   {: pre}

   Then encode the Gen 1 CRN and verify the version:

   ```bash
   ENCODED_CRN=$(jq -rn --arg v "$CLASSIC_CRN" '$v | @uri')
   curl -s -X GET \
     "https://api.${GEN2_REGION}.databases.cloud.ibm.com/v5/ibm/deployments/${ENCODED_CRN}" \
     -H "Authorization: Bearer $IAM_TOKEN" | jq '.deployment.version'
   ```
   {: pre}

3. **Verify and enable the Gen 1 private endpoint.**

   Check whether the Gen 1 source instance has a private endpoint enabled:

   ```bash
   curl -s -X GET \
     "https://resource-controller.cloud.ibm.com/v2/resource_instances/${CLASSIC_GUID}" \
     -H "Authorization: Bearer $IAM_TOKEN" | jq '.parameters.service_endpoints'
   ```
   {: pre}

   If private endpoints are not enabled on the Gen 1 instance, update the service instance:

   ```bash
   curl -s -X PATCH \
     "https://resource-controller.cloud.ibm.com/v2/resource_instances/${CLASSIC_GUID}" \
     -H "Authorization: Bearer $IAM_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"parameters": {"service-endpoints": "private"}}' | jq '{name: .name, state: .state}'
   ```
   {: pre}

   Replication cannot be set up unless the Gen 1 source has a private endpoint enabled.
   {: important}

4. **Provision the Gen 2 read replica.**

   The `resource_plan_id` for Gen 2 is `databases-for-postgresql-standard-gen2`. Pass the Gen 1 instance CRN in `parameters.dataservices.postgresql.read_replica.source_crn` with `source_type` set to `"gen1"`. Do not include `restore_backup_id`. That parameter is for the backup and restore path only.
   {: important}

   Optional sizing parameters (`version`, `host_flavor`, `members`, and `storage_gb`) are specified under `parameters.dataservices.postgresql`. If not provided, the service broker applies default values. The `storage_gb` value is specified per member in GB and must meet or exceed the data volume of the Gen 1 source deployment.

   ```bash
   PROVISION_RESPONSE=$(curl -s -X POST \
     "https://resource-controller.cloud.ibm.com/v2/resource_instances" \
     -H "Authorization: Bearer $IAM_TOKEN" \
     -H "Content-Type: application/json" \
     -d "{
       \"name\": \"$GEN2_NAME\",
       \"target\": \"$GEN2_REGION\",
       \"resource_group\": \"$RESOURCE_GROUP_ID\",
       \"resource_plan_id\": \"databases-for-postgresql-standard-gen2\",
       \"parameters\": {
         \"dataservices\": {
           \"postgresql\": {
             \"version\": \"18\",
             \"members\": 2,
             \"storage_gb\": 10,
             \"host_flavor\": \"bx3d.4x20\",
             \"read_replica\": {
               \"source_crn\": \"$CLASSIC_CRN\",
               \"source_type\": \"gen1\"
             }
           }
         }
       }
     }")
   echo "$PROVISION_RESPONSE" | jq '{name: .name, guid: .guid, state: .state}'
   ```
   {: pre}

   A read replica provisioning request carries the Gen 1 **instance** CRN in `dataservices.postgresql.read_replica.source_crn`. A backup and restore request carries a **backup** CRN in `dataservices.restore_backup_id`. These two parameters are not combined.
   {: note}

   Store the returned GUID for polling:

   ```bash
   GEN2_GUID=$(echo "$PROVISION_RESPONSE" | jq -r '.guid')
   ```
   {: pre}

5. **Wait until the instance is active.**

   Provisioning and initial data replication can take several hours depending on data volume. Poll the status until `state` shows `active`:

   ```bash
   curl -s -X GET \
     "https://resource-controller.cloud.ibm.com/v2/resource_instances/$GEN2_GUID" \
     -H "Authorization: Bearer $IAM_TOKEN" | jq '{state: .state, last_operation: .last_operation}'
   ```
   {: pre}

   Do not delete the Gen 1 instance while replication is in progress. The Gen 1 source instance must remain available until after the Gen 2 replica is promoted and cutover is complete.
   {: important}

#### Phase 2: Replicate, cut over, and finish
{: #api-rr-phase-2}
{: api}

1. **Configure a VPE gateway for private network access.**

   Gen 2 supports private endpoints only. Before switching applications, create a Virtual Private Endpoint (VPE) gateway from the [{{site.data.keyword.cloud_notm}} VPC networking console](https://cloud.ibm.com/infrastructure/provision/endpointGateway){: external} to connect to the new Gen 2 instance.

2. **Verify replication health and lag.**

   Check the replication status and verify that the replica is actively replicating data from the leader. You can query replication status from the Gen 1 leader database using `psql` (`SELECT * FROM pg_stat_replication;`).

3. **Stop writes to Gen 1.**

   Put your applications in maintenance mode or stop writes to the Gen 1 source instance to ensure all remaining transactions are replicated before promotion. Do not proceed while the replication lag is still high.
   {: important}

4. **Promote the Gen 2 read replica.**

   Retrieve the Gen 2 instance CRN, URL-encode it, and send a promotion request:

   ```bash
   GEN2_CRN=$(curl -s -X GET \
     "https://resource-controller.cloud.ibm.com/v2/resource_instances/$GEN2_GUID" \
     -H "Authorization: Bearer $IAM_TOKEN" | jq -r '.crn')
   ENCODED_GEN2_CRN=$(jq -rn --arg v "$GEN2_CRN" '$v | @uri')

   curl -s -X POST \
     "https://api.${GEN2_REGION}.databases.cloud.ibm.com/v5/ibm/deployments/${ENCODED_GEN2_CRN}/remotes/promotion" \
     -H "Authorization: Bearer $IAM_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"promotion": {}}' | jq '.task'
   ```
   {: pre}

   After promotion, the Gen 2 deployment becomes an independent primary database that accepts reads and writes, and automated Gen 2 backups are enabled. Promotion cannot be undone.
   {: note}

5. **Create service credentials.**

   Retrieve the Gen 2 instance CRN from the Resource Controller, then create a service credential with the `Manager` role:

   ```bash
   curl -s -X POST \
     "https://resource-controller.cloud.ibm.com/v2/resource_keys" \
     -H "Authorization: Bearer $IAM_TOKEN" \
     -H "Content-Type: application/json" \
     -d "{
       \"name\": \"${GEN2_NAME}-credentials\",
       \"source\": \"$GEN2_CRN\",
       \"parameters\": {
         \"role_crn\": \"crn:v1:bluemix:public:iam::::serviceRole:Manager\"
       }
     }" | jq '.credentials'
   ```
   {: pre}

6. **Update application endpoints.**

   Update your application to use the Gen 2 private endpoint from the credentials output instead of the Gen 1 endpoint.

   Gen 2 uses Let's Encrypt certificates rather than self-signed TLS certificates. Update any certificate validation or pinning in your application accordingly.
   {: note}

7. **Redeploy or restart applications.**

   Restart your applications with the updated endpoint and credentials. Confirm that they can read existing data and write new data on Gen 2 before proceeding.

8. **Finish and decommission Gen 1.**

   After validating that your applications are running correctly on Gen 2, proceed to [Post-migration tasks](#post-migration). Keep the Gen 1 deployment available through your planned retention period before decommissioning it.

### Backup and restore migration with the API
{: #api-backup-restore-migration}
{: api}

The backup and restore path creates a new Gen 2 instance from a backup of the Gen 1 source. All writes to Gen 1 after the backup are not included in the Gen 2 instance. Plan a maintenance window and stop application writes before taking a backup to ensure a consistent migration.

#### Phase 1: Capture backup and provision Gen 2
{: #api-br-phase-1}
{: api}

1. **Get the Gen 1 resource group ID and CRN.**

   Retrieve the resource group ID for the resource group where you plan to create the Gen 2 instance:

   ```bash
   curl -s -X GET \
     "https://resource-controller.cloud.ibm.com/v2/resource_groups?account_id=$ACCOUNT_ID" \
     -H "Authorization: Bearer $IAM_TOKEN" | jq '.resources[] | {name, id}'
   ```
   {: pre}

   Retrieve the Gen 1 instance CRN:

   ```bash
   CLASSIC_CRN=$(curl -s -X GET \
     "https://resource-controller.cloud.ibm.com/v2/resource_instances?name=$CLASSIC_NAME" \
     -H "Authorization: Bearer $IAM_TOKEN" \
     | jq -r '.resources[0].crn')
   echo "$CLASSIC_CRN"
   ```
   {: pre}

2. **Verify the PostgreSQL version on Gen 1.**

   Confirm that the source deployment is running PostgreSQL version 18 before proceeding. If the source is not on version 18, complete an in-place upgrade on Gen 1 first. For more information, see [Version requirement](#version-requirement).

   Set `GEN2_REGION` to the target Gen 2 region (for example, `us-east`):

   ```bash
   GEN2_REGION=<target-gen2-region>
   ```
   {: pre}

   Then encode the Gen 1 CRN and verify the version:

   ```bash
   ENCODED_CRN=$(jq -rn --arg v "$CLASSIC_CRN" '$v | @uri')
   curl -s -X GET \
     "https://api.${GEN2_REGION}.databases.cloud.ibm.com/v5/ibm/deployments/${ENCODED_CRN}" \
     -H "Authorization: Bearer $IAM_TOKEN" | jq '.deployment.version'
   ```
   {: pre}

3. **Stop writes to Gen 1 and take an on-demand backup.**

   For production migrations, stop application writes to ensure a consistent restore point, then trigger a backup. For development or test migrations where data completeness is not required, you can skip this step.

   ```bash
   curl -s -X POST \
     "https://api.${GEN2_REGION}.databases.cloud.ibm.com/v5/ibm/deployments/${ENCODED_CRN}/backups" \
     -H "Authorization: Bearer $IAM_TOKEN" \
     -H "Content-Type: application/json" | jq '.task'
   ```
   {: pre}

4. **Retrieve the backup CRN.**

   List backups and wait until one is completed and restorable:

   ```bash
   curl -s -X GET \
     "https://api.${GEN2_REGION}.databases.cloud.ibm.com/v5/ibm/deployments/${ENCODED_CRN}/backups" \
     -H "Authorization: Bearer $IAM_TOKEN" | jq '.backups[] | {id, status, is_restorable}'
   ```
   {: pre}

   Each backup's `id` is already the full backup CRN. Capture the most recent restorable backup automatically:

   ```bash
   BACKUP_CRN=$(curl -s -X GET \
     "https://api.${GEN2_REGION}.databases.cloud.ibm.com/v5/ibm/deployments/${ENCODED_CRN}/backups" \
     -H "Authorization: Bearer $IAM_TOKEN" \
     | jq -r '[.backups[] | select(.is_restorable == true)] | first | .id')
   echo "$BACKUP_CRN"
   ```
   {: pre}

5. **Provision the Gen 2 instance from the backup.**

   The `resource_plan_id` for Gen 2 is `databases-for-postgresql-standard-gen2` Specify `restore_backup_id` directly in `parameters.dataservices`; do not nest it under the service key. Optional service-specific sizing parameters, including `version`, `host_flavor`, `members`, and `storage_gb`, are specified in `parameters.dataservices.mongodb`. If these parameters are not provided, the service broker applies default values.

   The service shortname matches the product: `mysql`, `postgresql`, `redis`, `elasticsearch`, or `mongodb`.
   {: note}

   ```bash
   PROVISION_RESPONSE=$(curl -s -X POST \
     "https://resource-controller.cloud.ibm.com/v2/resource_instances" \
     -H "Authorization: Bearer $IAM_TOKEN" \
     -H "Content-Type: application/json" \
     -d "{
       \"name\": \"$GEN2_NAME\",
       \"target\": \"$GEN2_REGION\",
       \"resource_group\": \"$RESOURCE_GROUP_ID\",
       \"resource_plan_id\": \"databases-for-postgresql-standard-gen2\",
       \"parameters\": {
         \"dataservices\": {
           \"restore_backup_id\": \"$BACKUP_CRN\"
         }
       }
     }")
   echo "$PROVISION_RESPONSE" | jq '{name: .name, guid: .guid, state: .state}'
   ```
   {: pre}

   Store the returned GUID for polling:

   ```bash
   GEN2_GUID=$(echo "$PROVISION_RESPONSE" | jq -r '.guid')
   ```
   {: pre}

6. **Wait until the instance is active.**

   Restore can take several hours depending on data volume. Poll the status until `state` shows `active`:

   ```bash
   curl -s -X GET \
     "https://resource-controller.cloud.ibm.com/v2/resource_instances/$GEN2_GUID" \
     -H "Authorization: Bearer $IAM_TOKEN" | jq '{state: .state, last_operation: .last_operation}'
   ```
   {: pre}

   Do not delete the Gen 1 instance while the restore is running. Deleting the source also deletes its backups.
   {: important}

#### Phase 2: Configure access, cut over, and finish
{: #api-br-phase-2}
{: api}

1. **Configure a VPE gateway for private network access.**

   Gen 2 supports private endpoints only. Before switching applications, create a Virtual Private Endpoint (VPE) gateway from the [{{site.data.keyword.cloud_notm}} VPC networking console](https://cloud.ibm.com/infrastructure/provision/endpointGateway){: external} to connect to the new Gen 2 instance.

2. **Create service credentials.**

   Retrieve the Gen 2 instance CRN from the Resource Controller, then create a service credential with the `Manager` role:

   ```bash
   GEN2_CRN=$(curl -s -X GET \
     "https://resource-controller.cloud.ibm.com/v2/resource_instances/$GEN2_GUID" \
     -H "Authorization: Bearer $IAM_TOKEN" | jq -r '.crn')

   curl -s -X POST \
     "https://resource-controller.cloud.ibm.com/v2/resource_keys" \
     -H "Authorization: Bearer $IAM_TOKEN" \
     -H "Content-Type: application/json" \
     -d "{
       \"name\": \"${GEN2_NAME}-credentials\",
       \"source\": \"$GEN2_CRN\",
       \"parameters\": {
         \"role_crn\": \"crn:v1:bluemix:public:iam::::serviceRole:Manager\"
       }
     }" | jq '.credentials'
   ```
   {: pre}

3. **Update application endpoints.**

   Update your application to use the Gen 2 private endpoint from the credentials output instead of the Gen 1 endpoint.

   Gen 2 uses Let's Encrypt certificates rather than self-signed TLS certificates. Update any certificate validation or pinning in your application accordingly.
   {: note}

4. **Redeploy or restart applications.**

   Restart your applications with the updated endpoint and credentials. Confirm that they can read existing data and write new data on Gen 2 before proceeding.

5. **Finish and decommission Gen 1.**

   After validating that your applications are running correctly on Gen 2, proceed to [Post-migration tasks](#post-migration). Keep the Gen 1 deployment available through your planned retention period before decommissioning it.

## Post-migration tasks
{: #post-migration}

After the migration is complete, regardless of which path or interface you used:

* Validate application behavior and key business workflows on the Gen 2 deployment.
* Review performance, logs, and monitoring dashboards.
* Confirm that backup policies and alerting are configured on the Gen 2 deployment.
* Update runbooks, inventory records, and internal documentation to reflect the new Gen 2 endpoints and networking model.
* Delete migration-related service credentials that are no longer needed.
* Keep the Gen 1 deployment available through your planned retention period, as required by your rollback policy.
* Decommission the Gen 1 deployment only after validation and retention requirements are met.
