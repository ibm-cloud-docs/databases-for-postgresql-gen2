---
copyright:
  years: 2026
lastupdated: "2026-02-25"

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

For pricing estimation, use the **Add to estimate** button on the [{{site.data.keyword.databases-for-postgresql}} catalog page](https://cloud.ibm.com/catalog). Input your total consumption across two data members into the calculator. This is roughly double the size of your data because your data is replicated to both members. For example, 5 GB of disk with a 4 by 20 profile across two data members would be priced at 10 GB of disk and 8vCPU, 40 GB of RAM respectively. 

## Gen 2 backups pricing
{: #pricing-backup}

{{site.data.keyword.databases}} uses a snapshot based backup model, with pricing aligned to the size of your provisioned database storage. Snapshots differ from traditional backups in that they are block-level incremental copies, therefore you are billed based on how much data has changed since the last snapshot, not just the total size of your database. Snapshots have a minimum size of 1 GB and are rounded up to the next full Gigabyte.

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

* Day 1: A full snapshot is taken. Each snapshot captures the full volume, so you consume 200 GB of snapshot storage (100 GB × 2).

This worked example models the worst case scenario. The full snapshot is equal to the file system and in most cases, this will be smaller than the entire disk volume. New databases will be small and grow over time, therefore the first snapshot will be considerably smaller for new instances, reducing your overall bill for backups.
{: note}

* Day 2: You write 10 GB of new data to each member. The next snapshot is incremental — it only stores the changes since the last snapshot. So you consume an additional 20 GB (10 GB × 2 members), bringing your total snapshot usage to 220 GB.

* Day 30: You write 2 GB of data per day, bringing your total snapshot usage to ((2x28) + 220) = 276 GB.

* In a given month, if you have a {{site.data.keyword.databases-for-postgresql}} deployment that has 100 GB of disk per member, and have two data members, you receive 200 GB of snapshot storage free for that month. Your backup storage utilization is greater than 200 GB for the month (in this scenario), you are charged an overage of $0.095/month per gigabyte, therefore your total bill = (276 GB - 200 GB) X 0.095 = $7.22.

* With large instances and frequent writes, you’re more likely to exceed the free tier after the first snapshot, and your snapshot storage costs will grow quickly.

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
