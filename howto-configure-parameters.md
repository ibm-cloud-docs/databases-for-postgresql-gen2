---

copyright:
  years: 2026
lastupdated: "2026-09-10"

keywords: postgresql, databases, config, server parameters, configuration, gen2, max_connections, wal_level, shared_buffers, pgaudit, logical replication, postgresql parameters, changing configuration, updating parameters

subcollection: databases-for-postgresql-gen2

---

{{site.data.keyword.attribute-definition-list}}

# Changing your PostgreSQL configuration
{: #configure-parameters}

[Gen 2]{: tag-purple}

You can configure database parameters at provisioning time and on a running instance by using {{site.data.keyword.databases-for-postgresql_full}} Gen 2. Some parameters require a **rolling restart** (each member restarted one at a time); others are applied immediately with no downtime.

## Provisioning a new instance with custom parameters
{: #configure-parameters-provision}

Include only the parameters you want to set.

### Provisioning with the CLI
{: #configure-parameters-provision-cli}
{: cli}

```sh
ibmcloud resource service-instance-create <INSTANCE_NAME> databases-for-postgresql <SERVICE_PLAN_NAME> <REGION> \
  -p '{
    "dataservices": {
      "postgresql": {
        "configuration": {
          "max_connections": 300,
          "log_connections": true,
          "log_disconnections": true,
          "log_min_duration_statement": 500,
          "max_locks_per_transaction": 128,
          "max_prepared_transactions": 50,
          "wal_level": "logical",
          "shared_buffers": 196608,
          "max_worker_processes": 16,
          "max_logical_replication_workers": 8,
          "max_wal_senders": 20,
          "max_replication_slots": 20,
          "pgaudit.log": "ddl,role,read",
          "pgaudit.role": "auditor"
        }
      }
    }
  }'
```
{: pre}
{: cli}

### Provisioning with the API
{: #configure-parameters-provision-api}
{: api}

```sh
curl -X POST \
  https://resource-controller.cloud.ibm.com/v2/resource_instances \
  -H "Authorization: Bearer $IBMCLOUD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "<INSTANCE_NAME>",
    "resource_plan_id": "<SERVICE_PLAN_ID>",
    "resource_group": "<RESOURCE_GROUP_ID>",
    "target": "<REGION>",
    "parameters": {
      "dataservices": {
        "postgresql": {
          "configuration": {
            "max_connections": 300,
            "log_connections": true,
            "log_disconnections": true,
            "log_min_duration_statement": 500,
            "max_locks_per_transaction": 128,
            "max_prepared_transactions": 50,
            "wal_level": "logical",
            "shared_buffers": 196608,
            "max_worker_processes": 16,
            "max_logical_replication_workers": 8,
            "max_wal_senders": 20,
            "max_replication_slots": 20,
            "pgaudit.log": "ddl,role,read",
            "pgaudit.role": "auditor"
          }
        }
      }
    }
  }'
```
{: pre}
{: api}

## Updating parameters on a running instance
{: #configure-parameters-update}

Include only the parameters you want to change; all others are unchanged. Invalid parameter values are rejected with a 4xx error.

### Updating with the CLI
{: #configure-parameters-update-cli}
{: cli}

```sh
ibmcloud resource service-instance-update <INSTANCE_NAME_OR_CRN> \
  -p '{
    "dataservices": {
      "postgresql": {
        "configuration": {
          "max_connections": 300,
          "log_connections": true,
          "log_disconnections": true,
          "log_min_duration_statement": 500,
          "max_locks_per_transaction": 128,
          "max_prepared_transactions": 50,
          "wal_level": "logical",
          "shared_buffers": 196608,
          "max_worker_processes": 16,
          "max_logical_replication_workers": 8,
          "max_wal_senders": 20,
          "max_replication_slots": 20,
          "pgaudit.log": "ddl,role,read",
          "pgaudit.role": "auditor"
        }
      }
    }
  }'
```
{: pre}
{: cli}

### Updating with the API
{: #configure-parameters-update-api}
{: api}

```sh
curl -X PATCH \
  https://resource-controller.cloud.ibm.com/v2/resource_instances/<INSTANCE_ID> \
  -H "Authorization: Bearer $IBMCLOUD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "parameters": {
      "dataservices": {
        "postgresql": {
          "configuration": {
            "max_connections": 300,
            "log_connections": true,
            "log_disconnections": true,
            "log_min_duration_statement": 500,
            "max_locks_per_transaction": 128,
            "max_prepared_transactions": 50,
            "wal_level": "logical",
            "shared_buffers": 196608,
            "max_worker_processes": 16,
            "max_logical_replication_workers": 8,
            "max_wal_senders": 20,
            "max_replication_slots": 20,
            "pgaudit.log": "ddl,role,read",
            "pgaudit.role": "auditor"
          }
        }
      }
    }
  }'
```
{: pre}
{: api}

## Configuring PostgreSQL parameters
{: #configuring-parameters-pg}

### max_connections
{: #max_connections}

- **Type**: integer
- **Default**: 115
- **Range**: 25 – 500
- **Restart required**: Yes

Maximum concurrent client connections. The default of 115 reserves 15 connections for platform monitoring; 100 are available for client workloads.

### log_connections
{: #log_connections}

- **Type**: boolean
- **Default**: false
- **Restart required**: No

Logs each successful connection. Useful for compliance audit trails (SOC 2, PCI DSS).

### log_disconnections
{: #log_disconnections}

- **Type**: boolean
- **Default**: false
- **Restart required**: No

Logs each session at disconnect, including session duration.

### log_min_duration_statement
{: #log_min_duration_statement}

- **Type**: integer (milliseconds)
- **Default**: 100
- **Range**: -1 – 3,600,000
- **Restart required**: No

Logs statements exceeding the specified duration (ms). Set to -1 to disable, 0 to log all statements.

### max_locks_per_transaction
{: #max_locks_per_transaction}

- **Type**: integer
- **Default**: 64
- **Range**: 10 – 1,024
- **Restart required**: Yes

Average number of object locks available per transaction.

### max_prepared_transactions
{: #max_prepared_transactions}

- **Type**: integer
- **Default**: 0
- **Range**: 0 – 500
- **Restart required**: Yes

Maximum transactions in the prepared state. Set to 0 (default) to disable two-phase commit. Set to a non-zero value only if your application uses PREPARE TRANSACTION. When enabled, keep this value equal to max_connections.

### shared_buffers
{: #shared_buffers}

- **Type**: integer (8 kB blocks)
- **Default**: ~25% of host flavor RAM
- **Range**: ~25% RAM – ~40% RAM
- **Restart required**: Yes

Shared memory pool size in 8 kB blocks. 1 GiB = 131,072 blocks. The platform sets the default to ~25% of host RAM, with a maximum of ~40%. The value is automatically adjusted when infrastructure is scaled.

### wal_level
{: #wal_level}

- **Type**: enum
- **Default**: logical
- **Valid values**: replica, logical
- **Restart required**: Yes

Controls WAL verbosity. logical supports logical replication and CDC; replica supports physical replication only.

Reducing from logical to replica can make existing logical replication slots unusable. Ensure no CDC or logical replication workloads are active before changing this value.
{: .important}

### max_wal_senders
{: #max_wal_senders}

- **Type**: integer
- **Default**: 10
- **Range**: 10 – 40
- **Restart required**: Yes

Maximum simultaneous WAL sender processes. Each logical replication subscriber and physical standby requires one sender.

### max_replication_slots
{: #changing-configuration-max-replication-slots}

- **Type**: integer
- **Default**: 10
- **Range**: 10 – 40
- **Restart required**: Yes

Maximum replication slots. Each logical replication subscriber requires one slot. Drop unused slots promptly — abandoned slots retain WAL and can exhaust disk space. Must satisfy max_wal_senders >= max_replication_slots.

### max_worker_processes
{: #max_worker_processes}

- **Type**: integer
- **Default**: 8
- **Minimum**: 8
- **Maximum**: Memory-tiered. 8, 16, 24, or 32 depending on host flavor RAM
- **Restart required**: Yes

Total background worker budget, shared across logical replication and other background tasks.

| Host RAM | Maximum max_worker_processes |
|---|---|
| Less than 16 GiB | 8 |
| 16 – 31 GiB | 16 |
| 32 – 63 GiB | 24 |
| 64 GiB or more | 32 |
{: caption="max_worker_processes limits by RAM" caption-side="bottom"}

Must satisfy max_worker_processes >= max_logical_replication_workers + 4.

### max_logical_replication_workers
{: #max_logical_replication_workers}

- **Type**: integer
- **Default**: 4
- **Minimum**: 4
- **Maximum**: Memory-tiered — 4, 12, 20, or 28 depending on host flavor RAM
- **Restart required**: Yes

Workers available for logical replication. Maximum is max_worker_processes tier - 4:

| Host RAM | Maximum max_logical_replication_workers |
|---|---|
| Less than 16 GiB | 4 |
| 16 – 31 GiB | 12 |
| 32 – 63 GiB | 20 |
| 64 GiB or more | 28 |
{: caption="max_logical_replication_workers limits by RAM" caption-side="bottom"}

### pgaudit.log
{: #pgaudit_log}

- **Type**: string (comma-separated event classes)
- **Default**: "ddl"
- **Restart required**: No

Comma-separated list of statement classes to audit:

| Class | Statements logged |
|---|---|
| read | SELECT, COPY FROM |
| write | INSERT, UPDATE, DELETE, TRUNCATE, COPY TO |
| function | Function calls and DO blocks |
| role | GRANT, REVOKE, role and privilege changes |
| ddl | All DDL except role-related statements |
| misc | DISCARD, FETCH, CHECKPOINT, VACUUM, SET |
| misc_set | SET statements only |
| all | All of the previous statement classes |
| none | Disables session audit logging |
{: caption="pgaudit.log event classes" caption-side="bottom"}

Using all or write on busy instances can generate high log volumes.
{: .important}

### pgaudit.role
{: #pgaudit_role}

- **Type**: string
- **Default**: "" (empty — object-level audit logging disabled)
- **Restart required**: No

Role used for object-level audit logging. pgAudit logs activity on objects where this role has privileges. The role must exist before you set this parameter.
