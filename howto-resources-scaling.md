---

copyright:
  years: 2026
lastupdated: "2026-04-27"

keywords: postgresql, scaling, memory, disk IOPS, CPU, postgresql dedicated cores, scaling postgresql

subcollection: databases-for-postgresql-gen2

---

{{site.data.keyword.attribute-definition-list}}

# Adding disk, memory, and CPU
{: #resources-scaling}

[Gen 2]{: tag-purple}

To scale an [Isolated compute](/docs/cloud-databases?topic=cloud-databases-hosting-models&interface=cli#hosting-models-iso-compute-cli) host flavor instance, set the relevant `hostflavor` parameter to the Isolated Compute size you're targeting, such as "b3c.4x16.encrypted". As this includes vCPU and RAM allocation selections, do not separately select vCPU and RAM.
{: cli}

To scale an [Isolated compute](/docs/cloud-databases?topic=cloud-databases-hosting-models&interface=api#hosting-models-iso-compute-api) host flavor instance, set the relevant `host_flavor` parameter to the Isolated Compute size you're targeting, such as "b3c.4x16.encrypted". As this includes vCPU and RAM allocation selections, do not separately select vCPU and RAM.
{: api}

To scale an [Isolated compute](/docs/cloud-databases?topic=cloud-databases-hosting-models&interface=terraform#hosting-models-shared-compute-terraform) host flavor instance, set the relevant `host_flavor` parameter to the Isolated Compute size you're targeting, such as "b3c.4x16.encrypted". As this includes CPU and RAM allocation selections, do not separately select vCPU and RAM.
{: terraform}

You can manually adjust the resources available to your {{site.data.keyword.databases-for-postgresql_full}} deployment to suit your workload and the size of your data.

**Note: Terraform scaling allocations are per-member.**
{: terraform}

**Note: API scaling allocations use total allocation values.**
{: api}

## Resource breakdown
{: #resource-breakdown}

{{site.data.keyword.databases-for-postgresql}} deployments have two data members in a cluster, and resources are allocated to both members equally. For example, the minimum storage of a PostgreSQL deployment is 20840 MB, which equates to an initial size of 10240 MB per member. The minimum RAM for a PostgreSQL deployment per host and member is 20 GB.

Billing is based on the _total_ resources that are allocated to the service.
{: tip}

### Disk usage
{: #resources-scaling-disk-usage}

Storage shows the amount of disk space that is allocated to your service. Each member gets an equal share of the allocated space. Your data is replicated across all the data members in the PostgreSQL database cluster.

Disk allocation also affects the performance of the disk, with larger disks having higher performance. Baseline Input-Output Operations per Second (IOPS) performance for disk is 5 IOPS for each GB. Scale disk to increase the IOPS that your deployment can handle.

You cannot scale down storage. If your data set size has decreased, you can recover space by backing up and restoring to a new deployment.
{: tip}

### RAM
{: #ram-allocation}

If you find that your queries and database activity suffer from performance issues due to a lack of memory, you can scale the amount of RAM allocated to your service. If your database instance is on an Isolated Compute hosting model, select the vCPU x RAM configuration that matches your resource needs.

Adding memory to the total allocation adds memory to the members equally. {{site.data.keyword.databases-for-postgresql}} deployments have their memory allocation policy set at 50% heap and 50% system memory, so increasing the amount of RAM increases both heap and system memory. RAM can be scaled up or down.

[`work_mem`](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-WORK-MEM), [`maintenance_work_mem`](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-MAINTENANCE-WORK-MEM), and [`effective_cache_size`](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-EFFECTIVE-CACHE-SIZE) are auto-tuned based on the deployment's total memory. They are also set when you scale memory on your deployment. When you scale, the values are adjusted without outage to the running deployment.

The amount of memory allocated to the database's shared buffer pool is **not** adjusted automatically when you scale your deployment. Setting it to 25% of the deployment's total memory is recommended. You can manually tune the shared buffer pool through the [`shared_buffer`](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-SHARED-BUFFERS) setting in your [PostgreSQL's configuration](/docs/databases-for-postgresql?topic=databases-for-postgresql-changing-configuration). It is not auto-tuned because changing the `shared_buffer` requires a database restart.

### vCPU
{: #resources-scaling-cpu}

If you find that your database workloads need more vCPU resources, you can scale the amount of vCPU allocated to your service. Select the vCPU x RAM configuration that matches your resource needs.

## Scaling considerations
{: #resources-scaling-consider}

- Scaling up might cause your deployment to restart. If your deployment needs to be moved to a host with more capacity, the deployment is restarted as part of the move.
- Scaling down RAM or vCPU does not trigger restarts.
- Disk cannot be scaled down.
- Drastically scaling up vCPU, RAM, or disk can take longer to run than small resource increases to account for provisioning more underlying hardware resources.
- Scaling operations are logged in [{{site.data.keyword.atracker_full}}](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-at_events).
- If you find consistent trends in resource usage or want to scale when certain resource thresholds are reached, enable [autoscaling](/docs/databases-for-postgresql?topic=databases-for-postgresql-autoscaling) on your deployment.
- {{site.data.keyword.databases-for-postgresql}} is designed to balance work load across a cluster and can benefit from being horizontally scaled. If you are concerned about performance, check out [Adding PostgreSQL members](/docs/databases-for-postgresql?topic=databases-for-postgresql-horizontal-scaling).

## Review current resources and hosting model
{: #review-resources-ui}
{: ui}

In the **Resources** tab, you find both **Hosting model** and **Resource allocations** tiles. These tiles reflect your current resources and hosting model. Select *Configure* to adjust the settings in each tile.

## Scaling in the UI
{: #resources-scaling-ui}
{: ui}

In the **Resources** tab of the UI, select *Configure* on the **Resource allocations** tile. This opens up a panel where you can adjust your resources.

Look for a "Host sizes" table, where you can select the vCPU and RAM configuration per member for your database.

The "Disk (GB/member)" slider is your disk selection per member. Drag the slider or adjust the number in the input box to change the number of GB disk. Note that disk is tied to IOPS at 1 GB = 5 IOPS.

Members is the number of members of your database. For PostgreSQL, members are set to 2.

Review your total estimated cost in the calculator on the bottom. Note that if you have grandfathered costs, also known as legacy pricing structure, scaling your database instance removes some or all of your legacy pricing. For more information on grandfathering and when it ends, see [Gradfathering transition timeline](https://cloud.ibm.com/docs/cloud-databases?topic=cloud-databases-hosting-model-transition&interface=ui#hosting-model-transition-timeline-may25).

After you are done, click *Apply changes* to trigger the scaling operation.


## Review current resources and hosting model
{: #review-resources-cli}
{: cli}

To interact with {{site.data.keyword.databases-for}} on Gen 2 via the CLI you must utilize the IBM Cloud Resource Controller's CLI. For more information, see the [General IBM Cloud CLI (ibmcloud) commands](https://cloud.ibm.com/docs/cli?topic=cli-ibmcloud_cli). To get information about a particular instance, use the following command:

```sh
ibmcloud resource service-instance <INSTANCE_NAME> -o JSON
```
{: pre}


## Resources and scaling in the CLI
{: #resources-scaling-cli}
{: cli}

To update your instance (this includes operations, such as scaling and modifying other parts of your service), use the `ibmcloud resource service-instance-update` command.

```sh
ibmcloud resource service-instance-update <INSTANCE_NAME> -p '<{FIELDS_TO_UPDATE}>'
```
{: pre}

For example, to update the `host_flavor` of a {{site.data.keyword.databases-for-postgresql}} instance, use a command like:

```sh
ibmcloud resource service-instance-update test-database databases-for-postgresql standard us-south -p '{"host_flavor": "mx3d.8x80.encrypted", "storage_gb": 10 }'
```
{: pre}

### The `host_flavor` parameter
{: #host-flavor-parameter-cli}
{: cli}

Isolated Compute offers six size options to choose from.

The host_flavor parameter defines your Compute sizing. Input the appropriate value for your desired size.

| Host size | vCPU x RAM           | host_flavor value         |
|-----------|----------------------|---------------------------|
| 4x20      | 4 vCPU x 20 GB RAM   | bx3d.4x20.encrypted        |
| 8x40      | 8 vCPU x 40 GB RAM   | bx3d.8x40.encrypted        |
| 8x80      | 8 vCPU x 80 GB RAM   | mx3d.8x80.encrypted        |
| 16x80     | 16 vCPU x 80 GB RAM  | bx3d.16x80.encrypted       |
| 32x160    | 32 vCPU x 160 GB RAM | bx3d.32x160.encrypted      |
| 48x240    | 48 vCPU x 240 GB RAM | bx3d.48x240.encrypted      |
{: caption="Isolated Compute CLI selections" caption-side="bottom"}


## Review current resources and hosting model
{: #review-resources-api}
{: api}

The _Foundation endpoint_ that is shown on the _Overview_ panel of your service provides the base URL to access this deployment through the API. Use it with the `/groups` endpoint if you need to manage or automate scaling programmatically.

To view the current and scalable resources on a deployment, use the [/deployments/{id}/groups](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-api) endpoint.

```sh
curl -X GET https://api.{region}.databases.cloud.ibm.com/v5/ibm/deployments/{id}/groups -H 'Authorization: Bearer <>' \
```
{: pre}

To view the current and scalable resources on a deployment, use the [/deployments/{id}/groups](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-api) endpoint. Note that this command will also reveal if your database is a [Shared Compute](https://cloud.ibm.com/docs/cloud-databases?topic=cloud-databases-hosting-models&interface=ui#hosting-models-shared-compute-ui) or [Isolated Compute](https://cloud.ibm.com/docs/cloud-databases?topic=cloud-databases-hosting-models&interface=ui#hosting-models-iso-compute-ui) instance through the `host_flavor` attribute. If the `host_flavor` is null, it is on an old style hosting model.

```sh
curl -X GET -H "Authorization: Bearer $APIKEY" 'https://api.{region}.databases.cloud.ibm.com/v5/ibm/deployments/{id}/groups'
```
{: pre}

## Scaling with the API
{: #resources-scaling-api}
{: api}

To scale the `host_flavor` of a deployment to `mx3d.8x80.encrypted` for each memory member, use the following command:

```sh
curl -X POST https://resource-controller.cloud.ibm.com/v2/resource_instances
-H 'Authorization: Bearer <>'
-H 'Content-Type: application/json'
-d '{
   "name": "my-instance",
    "target": "ca-mon",
    "resource_group": "5c49eabc-f5e8-5881-a37e-2d100a33b3df",
    "resource_plan_id": "databases-for-postgresql-standard",
    "dataservices": {
      "resources": {
        "database": {
          "host_flavor": "mx3d.8x80.encrypted"
        }
      }
     }
   }'
```
{: pre}

For more information, see the [API reference](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-api).


### The `host_flavor` parameter
{: #host-flavor-parameter-api}
{: api}

Isolated Compute offers six size options to choose from.

The host_flavor parameter defines your Compute sizing. Choose the appropriate value for your desired size.

| Host size | vCPU x RAM           | host_flavor value         |
|-----------|----------------------|---------------------------|
| 4x20      | 4 vCPU x 20 GB RAM   | bx3d.4x20.encrypted        |
| 8x40      | 8 vCPU x 40 GB RAM   | bx3d.8x40.encrypted        |
| 8x80      | 8 vCPU x 80 GB RAM   | mx3d.8x80.encrypted        |
| 16x80     | 16 vCPU x 80 GB RAM  | bx3d.16x80.encrypted       |
| 32x160    | 32 vCPU x 160 GB RAM | bx3d.32x160.encrypted      |
| 48x240    | 48 vCPU x 240 GB RAM | bx3d.48x240.encrypted      |
{: caption="Isolated Compute API selections" caption-side="bottom"}


## Review current resources and hosting model
{: #review-resources-terraform}
{: terraform}

Review resource allocations to your database by checking your Terraform scripts for `cpu { allocation_count = }`, `memory {allocation_mb = }`, and `disk { allocation_mb = }`.

## Scaling with Terraform
{: #resources-scaling-terraform}
{: terraform}

Before executing a Terraform script on an existing instance, use the `terraform plan` command to compare the current infrastructure state with the desired state defined in your Terraform files. Any alteration to the `resource_group_id`, `service plan`, `version`, `key_protect_instance`, `key_protect_key`, `backup_encryption_key_crn` attributes recreates your instance. For a list of current argument references with the `Forces new resource` specification, see the [ibm_database Terraform Registry](https://registry.terraform.io/providers/IBM-Cloud/ibm/latest/docs/resources/database){: external}.
{: important}

Scale your instance by adjusting your Terraform script for the resource you're interested in. In the following example, `host_flavor` and `disk` allocations are specified.

To implement your change, run `terraform apply`.

```terraform
data "ibm_resource_group" "default" {
  name = "Default"
}
resource "ibm_resource_instance" "pg_demo" {
  name              = "pg-demo"
  service           = "databases-for-postgresql"
  plan              = "standard-gen2"
  location          = "ca-mon"
  resource_group_id = data.ibm_resource_group.default.id
  parameters_json = jsonencode(
    {
      "dataservices": {
        "postgresql": {
          "storage_gb": 9600,
          "members": 3,
          "host_flavor":  "bx3d.8x40"
        }
      }
    }
  )

```
{: codeblock}


## Switching to and Scaling Hosting Models in Terraform
{: #resources-switching-terraform}
{: terraform}

Select the [hosting model](/docs/cloud-databases?topic=cloud-databases-hosting-models) you want your database to be scaled to. You can change this later.

Update the `host_flavor` value in the parameters_json section from the example above with the intended value, then run `terraform apply`.
