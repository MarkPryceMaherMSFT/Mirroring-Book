# Chapter 9: Extended Capabilities

> **Part 1: Concepts and Architecture**
>
> **Purpose:** Use this chapter to assess Delta change data feed and Mirroring Views, including preview limitations and billing, before enabling them.

---

## Overview

Today, the extended capabilities are:

- **Delta change data feed** for row-level inserts, updates, and deletes
- **Mirroring Views** for replicating selected source views

Core mirroring copies tables into OneLake. Extended capabilities add optional preview features that consume extra compute and follow different operational rules.

## 9.1 Core Mirroring vs. Extended Capabilities

Before you enable anything, separate the default mirroring behaviour from the optional paid features.

| Core Mirroring (Included by Default) | Extended Capabilities (Optional, Paid) |
|---|---|
| Continuous replication of source tables into OneLake with standard mirrored database behaviour. | Optional preview features such as Delta change data feed and Mirroring Views. |

Core mirroring compute remains free. Extended capabilities are billed only for the extra work they perform.

---

## 9.2 Delta Change Data Feed

Delta change data feed, usually shortened to CDF, captures inserts, updates, and deletes at row level in the mirrored Delta tables. It is available for **all mirroring sources**.

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
| **Audit and compliance** | You need row-level change history. |
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

### Reading the Change Data Feed

To query CDF, first create a **Lakehouse shortcut** to the mirrored table and then read the shortcut from Spark. Direct CDF queries on the mirrored database item itself are not currently supported.

```python
# First create a Lakehouse shortcut to the mirrored table,
# then read the change data feed from the shortcut in Spark.
df = spark.read.format("delta") \
    .option("readChangeFeed", "true") \
    .option("startingVersion", 5) \
    .load("abfss://<workspace>@onelake.dfs.fabric.microsoft.com/<lakehouse>.Lakehouse/Tables/<shortcut_name>")

# The result includes _change_type, _commit_version, _commit_timestamp columns
df.show()
```

The change data feed output includes three important metadata columns:

| Column | Description |
|---|---|
| `_change_type` | Type of change: `insert`, `update_preimage`, `update_postimage`, or `delete` |
| `_commit_version` | Delta table version that contains the change |
| `_commit_timestamp` | Timestamp of the commit |

### Consuming CDF in Fabric Workloads

**Eventstreams connector:** A Fabric Eventstreams connector for mirroring CDF is in Preview. Check [the current Eventstreams documentation](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/event-streams/add-source-mirroring) for availability and configuration.

**Copy Job:** Direct Copy Job support for mirrored databases is in development. It is not currently available.

**Data Pipelines:** Use a Fabric Notebook activity within a Data Pipeline to run Spark code that reads from the CDF Lakehouse shortcut. A native Data Pipeline source connector for mirroring CDF is not currently available.

---

## 9.3 Mirroring Views

Mirroring Views replicates selected source views into OneLake as materialised Delta data.

### What It Does

- Replicates selected source views
- Stores the resulting data physically in OneLake
- Makes the view output available to downstream Fabric workloads

> **Note:** Mirroring Views is a Preview feature for **Snowflake only**.

> **Note:** Mirroring Views is a **paid extended capability** and follows the billing rules in Section 9.4.

### How It Works

Fabric reads the result set of each selected source view and writes it into OneLake as a materialised Delta table. This is different from normal mirrored table replication: view data is physically copied and refreshed on an approximately **12-hour** cycle rather than near real time.

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
| **Charging model** | Unified mirror-level charge | Unified mirror-level charge |

### What Is Charged

- Incremental replication work when changes occur
- Iterations that fail due to **user errors**
- Compute used to process inserts, updates, and deletes

### What Is Not Charged

- Core mirroring compute
- Storage for mirrored data
- Idle time with no changes
- Empty iterations with no data
- Iterations that fail due to **system errors**

### Unified Charging Rule

When Delta CDF and Mirroring Views are both enabled on the same mirrored database, Fabric applies a **single unified charge**. You are not billed twice.

[![Figure 9.3 - When extended capability billing applies](../assets/diagrams/chapter-09/diagram-03.png)](../assets/diagrams/chapter-09/diagram-03.excalidraw.png)
*Figure 9.3 - When extended capability billing applies*

### Capacity Requirement

A Fabric capacity is still required for setup and execution. Extended capability usage is charged through the **Mirror Replication Premium** operation on the `DataMovementIncrementalCopy` meter.

---

## Summary

Extended capabilities add optional preview features on top of core mirroring. Delta CDF supports change-aware downstream processing, while Mirroring Views brings selected Snowflake views into OneLake on an approximately 12-hour refresh cycle. Use both carefully, especially where preview status or billing sensitivity matters.

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 8: Using a Mirrored Database](chapter-08.md) | **Next:** [Chapter 10: Billing and Capacity Management](chapter-10.md)
