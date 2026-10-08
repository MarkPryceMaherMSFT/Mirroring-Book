# Chapter 9: Extended Capabilities

> **Part 1: Concepts and Architecture**
>
> **Purpose:** Use this chapter to assess Delta change data feed, Mirroring Views, and the announced Snowflake security-role replication preview, including their boundaries and billing.

**Part index:** [Chapters in Part 1](readme.md)

---

## Overview

The extended capabilities covered here are:

- **Delta change data feed** for row-level inserts, updates, and deletes
- **Mirroring Views** for replicating selected source views
- **Snowflake Security Roles Replication**, announced in Preview with rollout and configuration caveats below

Core mirroring copies tables into OneLake. Delta change data feed and Mirroring Views are optional paid extensions that consume extra compute and follow different operational rules. The newly announced security-role capability has a separate scope; its billing is not established by the CDF/views pricing below.

The [September 2026 FabCon feature summary](https://community.fabric.microsoft.com/blog/fbc_fabricupdatesblogs/fabric-september-2026-feature-summary/5325825#community-5325825-mcetoc_1k3kj91s5_130) explicitly announces **extended mirroring capabilities as generally available**, including Delta change feeds and source-view mirroring. It separately labels Snowflake security-role replication **Preview**. The [extended-capabilities overview](https://learn.microsoft.com/en-us/fabric/mirroring/extended-capabilities) and dedicated Views guide still carry preview labels when checked on **8 October 2026**. Use the announcement for its stated release status, but retain the source, refresh, consumption, and billing limits in the implementation guides.

**Snowflake security-role mirroring** is not included in the GA claim. Section 9.5 describes the announced role hierarchies, assignments, and grants, and the unresolved setup/availability boundary.

## 9.1 Core Mirroring vs. Extended Capabilities

Before you enable anything, separate the default mirroring behaviour from the optional paid features.

| Core Mirroring (Included by Default) | Extended Capabilities (Optional, Paid) |
|---|---|
| Continuous replication of source tables into OneLake with standard mirrored database behaviour. | Optional paid features such as Delta change data feed and Mirroring Views. |

Core mirroring compute remains free. Extended capabilities are billed only for the extra work they perform.

---

## 9.2 Delta Change Data Feed

Delta change data feed, usually shortened to CDF, records inserts, updates, and deletes at row level in the mirrored Delta tables. It is available across mirroring sources, including open mirroring partners. This is a feed of changes to the replicated Delta tables, not a replacement for source-side change capture. See [Delta change data feed in mirroring](https://learn.microsoft.com/en-us/fabric/mirroring/extended-capabilities-delta-change-data-feed).

**Related integration, separate controls:** Dataverse Link's low-latency synchronization also documents an optional Delta CDF, but that source-managed link is not the native mirrored database configured or billed in this chapter. Follow [Chapter 28](../Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-28.md#low-latency-sync-and-current-limits) for its own enablement, retention, and rollout boundaries.

### What It Does

- Records incremental row-level changes
- Supports downstream processing that reads only changed rows
- Works across all mirroring sources, including open mirroring sources

### How It Works

When CDF is enabled on a mirrored database, Fabric writes Delta change metadata alongside the mirrored table data in OneLake. Downstream Spark workloads can then read just the changed rows for a chosen version range.

[![Figure 9.1 - Delta change data feed flow](../assets/diagrams/chapter-09/diagram-01.png)](../assets/diagrams/chapter-09/diagram-01.excalidraw.png)
*Figure 9.1 - Delta change data feed flow*

### When to Use It

| Scenario | Description |
|---|---|
| **Incremental ETL** | Downstream jobs process only new or changed rows instead of re-reading full tables. |
| **Audit and compliance** | A downstream process persists row-level changes into a separately retained audit store. |
| **Event-driven processing** | Downstream logic reacts to specific data changes. |
| **Slowly changing dimensions** | Downstream models need change-aware history handling. |

### Enabling Delta Change Data Feed

CDF is enabled **per mirrored database**, not per table.

**Via the Fabric portal:**

1. Open the mirrored database.
2. Select the **gear** icon to open the configuration dashboard.
3. Under **Delta table management**, enable **Delta change data feed**.

**Via REST API:**

Use the mirrored database REST API. See the [API documentation](https://learn.microsoft.com/en-us/fabric/mirroring/mirrored-database-rest-api#enable-delta-change-data-feed-for-a-mirrored-database) for the request format.

Retrieve the current definition, add `enableDeltaChangeDataFeed: true` under `properties.target.typeProperties`, and update the full definition without losing existing settings. This API path also supports enabling CDF on existing mirrored tables.

### Reading the Change Data Feed

To query CDF, first create a **Lakehouse shortcut** to the mirrored table and then read the shortcut from Spark. Direct CDF queries on the mirrored database item itself are not currently supported. The following example uses a schema-enabled Lakehouse attached to the notebook:

```python
# First create a Lakehouse shortcut to the mirrored table,
# then read the change data feed from the shortcut in Spark.
df = spark.read.format("delta") \
    .option("readChangeFeed", "true") \
    .option("startingVersion", 5) \
    .table("<lakehouse>.<schema>.<shortcut_name>")

# The result includes _change_type, _commit_version, _commit_timestamp columns
df.show()
```

Replace version `5` with a retained version at or after CDF was enabled. CDF does not backfill earlier history, and its files are subject to VACUUM. It is not a permanent audit log. Persist required history downstream and keep consumers within the retention window. See [Delta CDF behaviour](https://learn.microsoft.com/en-us/fabric/data-engineering/delta-lake-change-data-feed) and [Delta table retention and VACUUM](chapter-04.md#delta-table-retention-and-vacuum).

The change data feed output includes three important metadata columns:

| Column | Description |
|---|---|
| `_change_type` | Type of change: `insert`, `update_preimage`, `update_postimage`, or `delete` |
| `_commit_version` | Delta table version that contains the change |
| `_commit_timestamp` | Timestamp of the Delta commit, not the original source transaction |

### Consuming CDF in Fabric Workloads

**Eventstreams connector:** The [September connector announcement](https://community.fabric.microsoft.com/blog/fbc_fabricupdatesblogs/fabric-september-2026-feature-summary/5325825#community-5325825-mcetoc_1k3kj91s5_62) places **Mirror DB Change Data Feed Connector under GA**. It reads changes directly from an actively syncing CDF-enabled mirror, including Open Mirroring. The [setup guide](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/event-streams/add-source-mirrored-database-change-feed) still says Preview, requires workspace Contributor or higher, and limits the current selection to **All tables**, raw events, and **no DeltaFlow transformations**. Its table-name/regex field description does not override the All tables restriction. Confirm supported options rather than treating the GA announcement as removal of those limits.

**Copy Job:** The summary announces **CDC and SCD Type 2 in Copy Job as GA**. For mirrored Delta CDF, keep the documented path **mirror → Lakehouse shortcut → Copy Job**; direct mirrored-database support is still described as in development. SCD2 maintains `Valid_From`, `Valid_To`, and `Is_Current`, but the [CDC guide](https://learn.microsoft.com/en-us/fabric/data-factory/cdc-copy-job) still labels SCD2 Preview and describes **net changes**, not every intermediate transaction. It is not a substitute for a separately retained, complete event audit.

**Eventstream and Copy Job integration:** The separately announced **Preview** source/destination integration does not make direct mirrored-database CDF consumption by Copy Job documented. Likewise, **Eventstream workspace Private Link is Preview**, and its [current support matrix](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/event-streams/set-up-tenant-workspace-private-links) does not explicitly list the mirrored-database change-feed connector. Do not promise that network/connector combination from the two announcements alone.

**Data Pipelines:** Use a Fabric Notebook activity within a Data Pipeline to run Spark code that reads from the CDF Lakehouse shortcut. A native Data Pipeline source connector for mirroring CDF is not currently available.

---

## 9.3 Mirroring Views

Mirroring Views replicates selected source views into OneLake as materialised Delta data.

### What It Does

- Replicates selected source views
- Stores the resulting data physically in OneLake
- Makes the view output available to downstream Fabric workloads

> **Note:** Mirroring Views supports **Snowflake only**. Its guide retains a preview label despite the general-availability release announcement described above.

> **Note:** Mirroring Views is a **paid extended capability** and follows the billing rules in Section 9.4.

### How It Works

Fabric reads the result set of each selected source view and writes it into OneLake as a materialised Delta table. This is different from normal mirrored table replication: view data is physically copied and refreshed on an approximately **12-hour** cycle rather than near real time.

These are source view results, not SQL view definitions deployed to the mirrored SQL endpoint. Complex views with nested subqueries or unsupported functions might not replicate successfully. See [Mirroring views](https://learn.microsoft.com/en-us/fabric/mirroring/extended-capabilities-views).

[![Figure 9.2 - Views and tables mirrored into OneLake](../assets/diagrams/chapter-09/diagram-02.png)](../assets/diagrams/chapter-09/diagram-02.excalidraw.png)
*Figure 9.2 - Views and tables mirrored into OneLake*

### When to Use It

| Scenario | Description |
|---|---|
| **Pre-aggregated reporting** | The source view already contains the required joins, filters, or aggregations. |
| **Source-defined shaping** | You want to preserve source-side logic without rebuilding it in Fabric immediately. |
| **Cross-table simplification** | A source view combines several tables into one analytical object. |

### Enabling Views

You can enable views in two places:

1. During creation of a new mirrored database
2. For an existing mirrored database through the configuration dashboard

When you enable views, Fabric asks you to acknowledge that extended capability billing applies.

---

## 9.4 Billing for Extended Capabilities

### Pricing Model

| Aspect | Delta Change Data Feed | Mirroring Views |
|---|---|---|
| **CU consumption rate** | 3 CU-hours | 3 CU-hours |
| **Meter** | `DataMovementIncrementalCopy` | `DataMovementIncrementalCopy` |
| **Operation name in billing** | Mirror Replication Premium | Mirror Replication Premium |
| **Billing scope** | Full mirror workload, including tables and any views | View processing when CDF is not enabled |

The published **3 CU-hours** rate is a usage rate, not a flat charge per database or refresh. Billing depends on actual work duration and throughput resources, with per-second granularity. Each active child job contributes when work runs in parallel. See [Billing for extended capabilities](https://learn.microsoft.com/en-us/fabric/mirroring/extended-capabilities-billing).

### What Is Charged

- Incremental replication work when changes occur
- Iterations that fail due to **user errors**
- Compute used to process inserts, updates, and deletes

### What Is Not Charged

- Core mirroring compute
- A separate extended-capability storage meter; the normal mirroring storage allowance and overage rules still apply
- Idle time with no changes
- Empty iterations with no data
- Iterations that fail due to **system errors**

### Unified Charging Rule

When Delta CDF and Mirroring Views are both enabled on the same mirrored database, Fabric applies a **single unified charge**. You are not billed twice.

This does not make the full mirror workload a fixed-price operation: CDF applies at mirror level, so its billing scope includes all replicated tables and any views. Additional CDF files increase storage consumption. Chapter 10 explains the capacity-based storage allowance and paused-capacity charges.

[![Figure 9.3 - When extended capability billing applies](../assets/diagrams/chapter-09/diagram-03.png)](../assets/diagrams/chapter-09/diagram-03.excalidraw.png)
*Figure 9.3 - When extended capability billing applies*

### Capacity Requirement

A Fabric capacity is still required for setup and execution. Extended capability usage is charged through the **Mirror Replication Premium** operation on the `DataMovementIncrementalCopy` meter.

---

## 9.5 Snowflake Security Roles Replication (Preview)

The [dedicated FabCon announcement](https://community.fabric.microsoft.com/blog/fbc_fabricupdatesblogs/fabric-september-2026-feature-summary/5325825#community-5325825-mcetoc_1k3kj91s5_131) describes an Extended Capability that brings supported **Snowflake role hierarchies, role assignments, and grants** into Fabric alongside the data. Its illustrated entry point is **Manage OneLake security**.

The feature summary says it will be available **shortly after FabCon EU 2026**, while the linked [OneLake companion announcement](https://community.fabric.microsoft.com/blog/fbc_fabricupdatesblogs/fabcon-and-sqlcon-barcelona-2026-what%E2%80%99s-new-in-microsoft-onelake-and-its-rapidly/5369146) says **now in public preview**. Neither statement proves availability in every tenant. The linked [extended-capabilities guide](https://learn.microsoft.com/en-us/fabric/mirroring/extended-capabilities) still provides no role-replication setup procedure, and the [Snowflake security guide](https://learn.microsoft.com/en-us/fabric/mirroring/snowflake-how-to-data-security) still requires separate Fabric security configuration.

Before adopting it, confirm the rollout and supported identity mapping, grant classes, hierarchy interpretation, revocation behavior, synchronization cadence, source privileges, and billing. Those details are not established by the linked announcement/setup pages. Do not infer support for every row-access or masking policy, or apply the CDF/views pricing table to role replication without its billing specification. Until the supported configuration is available and validated, continue managing Fabric access explicitly. See [Chapter 22](../Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-22.md#snowflake-security-roles-replication-preview).

## Summary

Extended capabilities add optional features on top of core mirroring. Delta CDF supports change-aware downstream processing, while Mirroring Views brings selected Snowflake views into OneLake on an approximately 12-hour refresh cycle; both have documented additional compute charges. The Snowflake role-replication Preview has a separate availability and configuration boundary. Check the source, consumer, security, and billing details rather than treating the GA announcement as universal feature parity.

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 8: Using a Mirrored Database](chapter-08.md) | **Next:** [Chapter 10: Billing and Capacity Management](chapter-10.md)
