# Chapter 8: Using a Mirrored Database

> **Part 1: Concepts and Architecture**
>
> **Purpose:** Use this chapter to query, secure, and combine mirrored data after replication is running.

***

## Overview

Mirrored data is a read-only analytical asset that you can query, join to other Fabric data, and expose to reporting tools.

[![Figure 8.1 - Analytics capabilities powered by a mirrored database](../assets/diagrams/chapter-08/diagram-01.png)](../assets/diagrams/chapter-08/diagram-01.excalidraw.png)
*Figure 8.1 - Analytics capabilities powered by a mirrored database*

> **Important:** Source-level security, including row-level security, column-level security, and data masking, is **not** propagated to the mirrored database in Fabric. Any granular security that existed in the source must be reconfigured separately in Fabric. See [Row-level security](https://learn.microsoft.com/en-us/fabric/data-warehouse/row-level-security), [Column-level security](https://learn.microsoft.com/en-us/fabric/data-warehouse/column-level-security), and [Dynamic data masking](https://learn.microsoft.com/en-us/fabric/data-warehouse/dynamic-data-masking) for how to apply these controls.

***

## SQL Analytics Endpoint

Every mirrored database exposes a **SQL analytics endpoint** for querying mirrored tables with T-SQL.

### Key Behaviours

* Mirrored tables are **read-only** through the SQL analytics endpoint.
* `INSERT`, `UPDATE`, and `DELETE` statements against mirrored tables are not supported.
* You **can** create SQL views and stored procedures on the SQL analytics endpoint. These are metadata-only objects and do not write to the mirrored tables.
* Metadata synchronisation can lag behind the underlying Delta tables. If a newly mirrored table does not appear, use the **Refresh** action from the SQL analytics endpoint item menu.

> **Note:** Query freshness through the SQL analytics endpoint depends on a second background process, often called **metadata sync**, not only on mirroring replication speed. Metadata sync keeps the endpoint's SQL view of the Delta tables current, detecting new or dropped tables, schema changes, and row-level data changes. Because it runs separately from mirroring, a short additional delay is possible between a change landing in the mirrored Delta tables and that change becoming visible through the SQL analytics endpoint. If you need the current state without waiting on sync, query through Spark or Power BI DirectLake instead, since both read the Delta files directly. See [SQL analytics endpoint metadata sync](https://learn.microsoft.com/en-us/fabric/data-engineering/sql-analytics-endpoint-metadata-sync) for how sync works, the newer low-latency metadata sync option in preview, and manual refresh options through the portal, REST API, or a T-SQL stored procedure.

### String Column Size Boundary

String columns support up to 16 MB per value, equivalent to `varchar(max)`. Tables created under an earlier limit (8000 bytes) may need to be recreated. See the [Fabric Mirroring troubleshooting page](https://learn.microsoft.com/en-us/fabric/mirroring/troubleshooting) for current guidance.

***

## Openness of the Delta Format

Mirrored data is stored as **Delta** tables in OneLake.

**What this means in practice:**

* Tools that can read Delta or Parquet can access mirrored data in OneLake.
* OneLake exposes an [ADLS Gen2-compatible API](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction), so tools such as Azure Databricks, dbt, Trino, Presto, and Apache Spark can connect directly.
* You do not need to export mirrored data into another format before analysing it.
* The format remains open even though the mirrored tables themselves stay read-only.

***

## Cross-Database Queries

The SQL analytics endpoint supports **cross-database queries**, so you can join mirrored data with warehouses, lakehouse SQL endpoints, and other SQL-addressable Fabric items.

### Joining Mirrored and Warehouse Data

```sql
SELECT
    m.OrderID,
    m.OrderDate,
    m.TotalAmount,
    w.CustomerSegment
FROM
    [SalesMirror].[dbo].[Orders] AS m
INNER JOIN
    [SalesWarehouse].[dbo].[CustomerSegments] AS w
    ON m.CustomerID = w.CustomerID
WHERE
    m.OrderDate >= '2024-01-01';
```

### Three-Part Naming Convention

Cross-database queries use the form `[database].[schema].[table]`. The database name is the Fabric item name, whether that item is a mirrored database, warehouse, or lakehouse SQL endpoint.

***

## Shortcuts to Mirrored Data

Fabric **Shortcuts** let you reference mirrored tables from a Lakehouse without copying the data.

### Creating a Shortcut to Mirrored Data

1. Open a Fabric Lakehouse.
2. In **Tables**, select **+ New shortcut**.
3. Choose **Microsoft OneLake** as the source.
4. Browse to the mirrored database path and select the required table.

The shortcut then appears as a Lakehouse table that Spark and the Lakehouse SQL endpoint can query.

### Use Cases for Shortcuts

* Reading mirrored data in Spark notebooks
* Combining mirrored tables with Lakehouse-managed tables
* Preparing downstream transformations without duplicating storage

***

## Tool Ecosystem

Mirrored databases can be consumed through the SQL analytics endpoint and through OneLake. In every case, mirrored tables remain read-only. No tool can write to the mirrored Delta tables.

### SQL-Based Tools

| Tool                                    | Connection Method           | Notes                                    |
| --------------------------------------- | --------------------------- | ---------------------------------------- |
| **SQL Server Management Studio (SSMS)** | TDS / SQL Server connection | Read-only access to mirrored tables      |
| **Azure Data Studio**                   | TDS / SQL Server connection | Querying and notebook support            |
| **DBeaver**                             | JDBC / SQL Server driver    | Community tool with broad compatibility  |
| **Power BI Desktop**                    | DirectQuery or Import       | Uses Fabric and SQL connectivity options |
| **Excel**                               | Get Data -> SQL Server      | Can read mirrored tables directly        |
| **Tableau**                             | SQL Server connector        | Read-only analytics on mirrored data     |

### Spark / Notebook Tools

| Tool                 | Connection Method      | Notes                           |
| -------------------- | ---------------------- | ------------------------------- |
| **Fabric Notebooks** | Native OneLake / Spark | Read Delta data directly        |
| **Azure Databricks** | OneLake ABFS connector | Reads Delta tables from OneLake |
| **Apache Spark**     | ADLS Gen2 API          | Direct Delta table access       |

***

## Power BI Integration

Power BI is a common way to consume mirrored data in Fabric.

### DirectLake Mode

Power BI uses **DirectLake mode** when the semantic model is built over the Delta Parquet files in OneLake. The SQL analytics endpoint is not used in this path.

[![Figure 8.2 - DirectLake mode reads Delta files directly from OneLake](../assets/diagrams/chapter-08/diagram-02.png)](../assets/diagrams/chapter-08/diagram-02.excalidraw.png)
*Figure 8.2 - DirectLake mode reads Delta files directly from OneLake*

* DirectLake reads the Delta Parquet files directly from OneLake.
* It avoids the SQL endpoint during query execution.
* It does not require a scheduled import refresh.
* It reflects newly replicated data as the semantic model sees updated files.

### Creating a Power BI Semantic Model from a Mirrored Database

1. Open the mirrored database item in Fabric.
2. Select **Create semantic model**.
3. Choose the tables to include.
4. Let Fabric create the semantic model in the same workspace.
5. Build reports on top of that model.

### DirectQuery Mode

If DirectLake is unsuitable for a particular model, Power BI can use **DirectQuery** through the SQL analytics endpoint. In that case, queries are executed by the SQL engine rather than directly against OneLake files.

### Refreshing Imported Models

If you choose **Import** mode, schedule refreshes as needed. Mirroring does not need to stop for an import refresh to succeed.

***

## Summary

A mirrored database in Fabric is primarily a read-only analytical asset. You can query it through the SQL analytics endpoint, reference it through shortcuts, join it across databases, and use it in Power BI, while keeping source security and Fabric-side security responsibilities clearly separated.

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 7: Deploying a Mirrored Database Using CI/CD](chapter-07.md) | **Next:** [Chapter 9: Extended Capabilities](chapter-09.md)
