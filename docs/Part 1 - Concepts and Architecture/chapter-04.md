# Chapter 4: The Anatomy of a Mirrored Database

> **Part 1: Concepts and Architecture**
>
> **Purpose:** This chapter helps you understand what Fabric creates when you mirror a source, where the replicated data is stored, how the replicator behaves, and what operational details matter after setup.

***

## Overview

A mirrored database in Fabric is not just a copied set of tables. It is a managed structure that combines storage, replication state, and query surfaces inside OneLake.

When you create a mirrored database, Fabric creates **two** items:

1. The **mirrored database item** itself, which owns the landing zone, Delta tables, replicator engine, replication settings, and monitoring state.
2. The **SQL analytics endpoint**, which exposes the mirrored tables through a read-only SQL surface.

Conceptually, you can think of the mirrored database item as the ingestion and storage layer, which can be the source of a Fabric shortcut or queried using a Delta reader, and the SQL analytics endpoint as the query layer that sits over the resulting Delta tables.

***

## 4.1 Storage Architecture: Landing Zone and Delta Tables

[![Figure 4.1: Storage architecture from landing zone to Delta tables](../assets/diagrams/chapter-04/diagram-01.png)](../assets/diagrams/chapter-04/diagram-01.excalidraw.png)
*Figure 4.1: Storage architecture from landing zone to Delta tables*

### The Landing Zone

The **landing zone** is a transient staging area inside the mirrored database item. Change data arrives here before Fabric applies it to the final table layer.

**Key points:**

* The landing zone is Fabric-managed and is not intended as a user-facing storage location. *(Except for Open Mirroring)*
* The exact file pattern depends on the source and mirroring method. (These are hidden from the user)
* Retention and cleanup behaviour depend on the mirroring type and source.
* Replicator engine moves processed files to an internal cleanup folder and removes them after **7 days**, while retaining the latest processed data file.

### Delta Tables

After Fabric processes the landing zone files, the replicated data is written into **Delta tables** in OneLake.

**Key points:**

* The table layer is stored in **Delta** format.
* Delta transaction logs preserve table state and change application order.
* This is the main queryable data layer for SQL, Spark, and DirectLake scenarios. 
* Anything that can read/understand Delta can query the data.
* Change Data Feed (CDF) can be enabled for an additional cost.
* The mirrored database item owns these tables even though users often access them through the SQL analytics endpoint.

***

## 4.2 The Replicator

The **replicator** is the Fabric service that manages extraction, landing zone writes, and application of changes into Delta tables.

### Replicator Responsibilities

1. **Source connectivity:** Maintains the connection to the source by using the Fabric connection or the supported source-side integration.
2. **Change extraction:** Reads new and changed data through the source's supported mechanism.
3. **Change position tracking:** Maintains watermarks and LSN offsets to track the last processed change position.
4. **Landing zone writes:** Writes the captured changes into the landing zone.
5. **Delta application:** Applies inserts, updates, and deletes into the Delta table layer.
6. **Retry management:** Handles transient failures and backoff behaviour.

### Replicator Lifecycle

[![Figure 4.2: Replicator state machine and lifecycle](../assets/diagrams/chapter-04/diagram-02.png)](../assets/diagrams/chapter-04/diagram-02.excalidraw.png)
*Figure 4.2: Replicator state machine and lifecycle*

This is a conceptual model of what the replicator does internally. It does not map one-to-one to the replication status values returned by the Fabric portal or the REST API, covered in Chapter 5.

* **Initial snapshot:** Fabric performs a full load of the selected objects.
* **Incremental replication:** Fabric switches to change processing after the snapshot completes.
* **Retrying (backoff):** Fabric waits and retries after transient issues, without leaving incremental replication.
* **Stopped:** Replication has been halted by an operator.
* **Failed:** Replication cannot continue without intervention.

> **Note:** For database mirroring sources, stopping mirroring and then starting it again triggers a **full reseed** from the initial snapshot. It does not resume from the previous watermark.

***

## 4.3 Configuration Screens and Management

Mirrored databases are created and managed in the Fabric portal, REST API or via the deployment process.

### Create a Mirrored Database \[in the UX]

1. Open the target workspace.
2. Select **New item** and choose the relevant mirrored database source.
3. Select or create the required Fabric connection.
4. Choose the schemas or tables to mirror.
5. Start mirroring to begin the initial snapshot.

### Common Configuration Options

| Option               | Description                                              |
| -------------------- | -------------------------------------------------------- |
| **Connection**       | The Fabric connection that stores source access details. |
| **Tables to mirror** | The selected tables or schemas included in the mirror.   |
| **Start mirroring**  | Starts the initial snapshot or reseed process.           |
| **Stop mirroring**   | Stops active replication.                                |

### Management Notes

* You can monitor table status, lag, and errors from the mirrored database item.
* You can add or remove mirrored tables, subject to connector support.
* For database mirroring sources, **Stop mirroring** followed by **Start mirroring** triggers a **full reseed** from the initial snapshot rather than a resume from the previous watermark.

***

## 4.4 SQL Analytics Endpoint

Every mirrored database also creates a **SQL analytics endpoint**. This is the read-only SQL surface over the mirrored Delta tables.

**Key points:**

* The endpoint is created automatically with the mirrored database item.
* It exposes the mirrored tables through T-SQL.
* It is read-only for mirrored data.
* It can be used from tools such as SQL Server Management Studio, Azure Data Studio, Power BI, and other SQL clients.
* The mirrored database item and the SQL analytics endpoint are separate Fabric items, even though users often move between them during normal work.

> **Note:** The SQL analytics endpoint metadata sync can lag behind the data arriving in the Delta tables. If newly mirrored tables do not appear, use the **Refresh** action in the SQL analytics endpoint context menu. A new metadata sync option is in Public Preview; see [SQL analytics endpoint metadata sync](https://learn.microsoft.com/en-us/fabric/data-engineering/sql-analytics-endpoint-metadata-sync) for details.
>
> As of writing, you can use the new Preview feature of MD Sync v2. This has a significantly improved implementation, so the latency challenges you may see in MD Sync v1 are no longer a problem.
>
> The SQL analytics endpoint is a whole book in its own right. It is the same engine used by the Fabric Data Warehouse.

***

## 4.5 Security of a Mirrored Database

Fabric mirrored databases use the Fabric security model for access to replicated data, but source-side controls do not automatically carry over.

[![Figure 4.3: Security model for mirrored databases](../assets/diagrams/chapter-04/diagram-03.png)](../assets/diagrams/chapter-04/diagram-03.excalidraw.png)
*Figure 4.3: Security model for mirrored databases*

### Workspace and Item Access

* Workspace roles control who can create, manage, and view mirrored database items.
* Item permissions control who can access the mirrored database item and its SQL analytics endpoint.

### OneLake Security for Mirrored Databases (Preview)

Workspace roles and item permissions, covered above, are coarse-grained. They control whether someone can open or manage the mirrored database item at all. **OneLake security** is Fabric's finer-grained, data-plane security model. It controls which tables, folders, rows, or columns inside that item a user can actually read, and it is enforced consistently across every engine that reads the data, including SQL, Spark, Power BI, and authorized third-party engines.

**What it adds for mirrored databases:**

* OneLake security roles can now be defined directly on **mirrored database items**, for all mirroring types. This capability is in **Preview**.
* Each role has four parts: the **data** it covers (specific tables or folders), the **permission** it grants, its **members** (the users or groups assigned to it), and any **constraints** that exclude specific rows or columns from the role.
* You enable and manage these roles from the mirrored database item's own experience in the Fabric portal, the same pattern already used for lakehouses.
* Workspace **Admins** and **Members** can create and manage OneLake security roles.

**Who is affected:**

* OneLake security roles apply to users who have the **Viewer** workspace role, or **Read** item permission on the mirrored database. Those users only see the tables or folders their assigned role grants.
* Workspace **Admins**, **Members**, and **Contributors** bypass OneLake security roles entirely. They can read and write all data in the item regardless of role membership.
* A **DefaultReader** role exists by default and grants access to any user holding the **ReadAll** item permission. You can edit or delete it to tighten default access.

**Shortcuts to mirrored data:**

* A shortcut created against a mirrored table automatically respects the OneLake security defined on the source mirrored item. You do not need to duplicate or re-apply security when other users reach the data through a shortcut, which makes it safe to share mirrored data broadly without duplicating it.

> **Note:** OneLake security is separate from the SQL-layer object permissions, row-level security, and column-level security described next. OneLake security is enforced at the OneLake data-plane level and applies no matter which engine reads the data. SQL-layer security applies only to queries made through the SQL analytics endpoint.

For setup steps, see [Get started with OneLake security](https://learn.microsoft.com/en-us/fabric/onelake/security/get-started-security).

### SQL-Layer Security

* Object permissions, row-level security, and column-level security can be configured on the SQL analytics endpoint where supported.
* These controls are configured in Fabric. They are not copied from the source system.

> **Important:** Source-level security settings, such as row-level security, column-level security, and data masking, are **not** propagated to the Fabric mirrored database. Any granular security must be reconfigured in Fabric after mirroring is set up. See [Security in the Fabric Data Warehouse documentation](https://learn.microsoft.com/en-us/fabric/data-warehouse/sql-granular-permissions) for how to apply these controls.

### Credentials and Identity

* Source credentials are stored in **Fabric connections**.
* Authentication paths may use **Microsoft Entra ID**, service principals, or other connector-specific methods.
* End users of the mirrored database do not need direct access to the stored source credentials.

### Encryption Notes

* Data in transit uses TLS.
* Data at rest in OneLake is encrypted by Fabric.
* **Azure Cosmos DB mirroring does not support customer-managed keys on OneLake.** Refer to the [Azure Cosmos DB mirroring limitations](https://learn.microsoft.com/en-us/fabric/mirroring/azure-cosmos-db-limitations) for current guidance.

***

## Summary

A mirrored database in Fabric consists of a mirrored database item, a SQL analytics endpoint, a landing zone, Delta Parquet tables, and the replicator state that keeps those layers moving. Understanding these parts makes it easier to plan retention, diagnose reseeds, manage endpoint visibility, and apply security correctly inside Fabric.

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 3: Methods of Mirroring: Push, Pull or Polling, and Shortcuts](chapter-03.md) | **Next:** [Chapter 5: Monitoring a Mirrored Database](chapter-05.md)
