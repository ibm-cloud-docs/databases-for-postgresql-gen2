---
copyright:
  years: 2026
lastupdated: "2026-04-14"

keywords: postgresql, gen 2, pricing

subcollection: databases-for-postgresql-gen2

---

{{site.data.keyword.attribute-definition-list}}

# Pricing
{: #pricing}

[Gen 2]{: tag-purple}

A {{site.data.keyword.databases-for-postgresql}} deployment consists of a highly available PostgreSQL cluster with two data members, ensuring your data is replicated across both. Pricing is based on the total resources allocated to the deployment-disk storage, RAM, virtual CPU cores, and backup storage—calculated on an hourly prorated basis. Gen 2 instances require at least 5 GB of disk space, and the smallest configuration profile offers 4 vCPU cores.

## Using the pricing calculator
{: #pricing-calc}

For pricing estimation, use the **Add to estimate** button on the [{{site.data.keyword.databases-for-postgresql}} catalog page](https://cloud.ibm.com/catalog). Input your total consumption across two data members into the calculator. This is equal to the number of members because your data is replicated to all members. For example, 5 GB of disk on a 4 vCPU x 20 GB RAM profile would have a total bill for 10 GB of disk and the total cost of 2 members.

## Gen 2 backups pricing
{: #pricing-backup}

Gen 2 {{site.data.keyword.databases-for}} uses a snapshot based backup model, with pricing aligned to the size of your provisioned database storage. Snapshots differ from traditional backups in that they are block-level incremental copies, therefore you are billed based on how much data has changed since the last snapshot, not just the total size of your database. Snapshots have a minimum size of 1 GB and are rounded up to the next full Gigabyte.

By default, {{site.data.keyword.databases-for-postgresql}} provides a daily backup that is stored for 30 days. These backups, and any on-demand backups you make, all count toward the above allocation.

Backup storage included:

* You receive free backup storage equal to the total provisioned disk size of your deployment.
* This includes both automated daily backups and manual (on-demand) snapshots.
* Example: If your 2 member {{site.data.keyword.databases-for-postgresql}} deployment is provisioned with 100 GB of disk per member, you get 200 GB of backup storage included at no cost.

Overage charges:

* The overage is billed monthly.
* Total snapshot storage = Day 1 full + (Daily change × 29 days x number of members)
* Overage is charged at $0.095 per GB per month.

Worked example, for a 2-member PostgreSQL deployment with 100 GB of data per member:

* Day 1: A full snapshot is taken from the current primary. This consumes 100 GB of snapshot storage.

This models the worst case scenario where the full snapshot is equal to the file system size. In practice, especially for new databases that grow over time, snapshot sizes are typically smaller, which helps reduce backup costs.
{: note}

* Day 2-16: You write 10 GB of new data per day. Snapshots are incremental and only store changes. Over 15 days, this adds 150 GB, bringing total snapshot usage to 100 GB + 150 GB = 250 GB.

* Your backup storage utilization is now greater than the free allocation of 200 GB for the month (in this scenario). Billing incurs for an overage at a rate of $0.095/month per gigabyte.

* Day 17: A failover occurs, and one secondary member becomes the new primary. A full snapshot is taken from this new primary, consuming another 100 GB.

* Day 18-30: You continue writing 10 GB per day, adding 130 GB over 13 days.
  
Total snapshot = 100 GB (initial) + 150 GB (incremental) + 100 GB (failover snapshot) + 130 GB (post-failover incremental) = 480 GB
Free allocation = 100 GB x 2 members = 200 GB
Overage = 480 GB - 200 GB = 280 GB
Monthly charge = (480 GB - 200 GB) x $0.095 = $26.6

With large deployments and frequent writes, you may exceed the free tier after the first snapshot.

* Cross-region copies: If you choose to copy snapshots to another region, {{site.data.keyword.cloud}} charges for the full size of the snapshot in the destination region (not incremental) and continued incremental growth in the original region as new snapshots are taken.

## Scaling per member
{: #scaling-member}

{{site.data.keyword.databases-for-postgresql}} instances have minimum and maximum allocation for disk and RAM as shown. Scaling instances through the API and CLI provides more granularity and also allows you to scale a database instance up to 9600 GB of disk per member. Minimum and maximum CPU and RAM combinations vary per region and as per the host flavor, see [Isolated Compute](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-isolated-compute&interface=cli).

| Resource | Minimum | Maximum | Scaling granularity (API/CLI) |
| ---------- | ----- | ----- | ------- |
| Disk | 10 GB per member | 9600 GB per member | 1024 MB per member |
| RAM | 16 GB | 240 GB | Isolated Compute – Resource scaling via T-shirt sizes |
| CPU | 4 vCPU | 48 vCPU| Isolated Compute – Resource scaling via T-shirt sizes |
{: caption="Scaling limits" caption-side="top"}
