# Chapter 3: Methods of Mirroring: Push, Pull or Polling, and Shortcuts

> **Part 1: Concepts and Architecture**
>
> **Purpose:** This chapter helps you identify how each mirrored source moves or exposes data so you can plan connectivity, latency, storage, and operations before you configure a mirror.

***

## Overview

In this chapter, **push**, **pull or polling**, and **shortcuts** are terms used to compare how Fabric mirroring behaves across sources. They are **not** an official Microsoft product taxonomy. The connector for each source determines the method. You do not choose it during setup.

> You don't *need* to read this chapter, but if you really want to understand Mirroring, the concept of how data is moved from the source system to the landing zone is really important.

These three methods answer practical questions:

* Who initiates the movement or exposure of data?
* Is data copied into OneLake, or read in place?
* What level of latency should you plan for?
* What connectivity and operational model does the source require?

| Method                    | What happens                                                | Data copied into OneLake? | Main replication concern                                |
| ------------------------- | ----------------------------------------------------------- | ------------------------: | ------------------------------------------------------- |
| **Push-based**            | The source-side mechanism sends changes towards Fabric.     |                       Yes | Source connectivity and source-side change capture      |
| **Pull or polling-based** | Fabric polls the source for changes and ingests them.       |                       Yes | Polling behaviour, connectivity, and replication lag    |
| **Shortcuts**             | Fabric synchronises metadata and creates OneLake shortcuts. |                        No | Query path, source storage access, and metadata refresh |

> It is possible to combine push and pull Mirroring at the same time, Snowflake Mirroring is one of these rare examples.

[![Figure 3.1: The three methods overview](../assets/diagrams/chapter-03/diagram-01.png)](../assets/diagrams/chapter-03/diagram-01.excalidraw.png)
*Figure 3.1: The three methods overview*

***

## Push-Based Mirroring

In the replication models used in this book, **push-based** mirroring means the source-side technology captures changes and sends them towards Fabric without Fabric repeatedly polling for each batch of changed rows.

For Fabric database mirroring, the underlying mechanism depends on the source:

* **Azure SQL Database** uses the **Fabric mirroring change feed**.
* **Azure SQL Managed Instance** on the **Always-up-to-date** or **SQL Server 2025** update policy uses the **Fabric mirroring change feed**.
* **SQL Server 2025** uses the **Fabric mirroring change feed + Azure Arc + data gateway**.
* **Azure Cosmos DB** uses **Continuous Backup**, which continuously captures inserts, updates, and deletes without Fabric needing to poll for changes.
* **Fabric SQL Database** uses the **Fabric mirroring change feed**, auto-configured inside Fabric.

[![Figure 3.2: Push-based sequence](../assets/diagrams/chapter-03/diagram-02.png)](../assets/diagrams/chapter-03/diagram-02.excalidraw.png)
*Figure 3.2: Push-based sequence*

### How It Works

1. A source-side mechanism captures committed changes.
2. The source or its paired component sends those changes to the Fabric landing zone.
3. Fabric detects the new landing zone files.
4. Fabric applies the change batches into Delta tables in OneLake.
5. Fabric manages offset tracking so it knows the last processed change position.

### Characteristics

| Attribute                 | Detail                                                                                          |
| ------------------------- | ----------------------------------------------------------------------------------------------- |
| **Change initiator**      | Source-side mechanism                                                                           |
| **Latency**               | Very low, often seconds to low minutes depending on source, workload, and processing conditions |
| **Agent required**        | Depends on source. Some paths are native to the platform; i.e. Azure Arc components             |
| **Outbound connectivity** | Usually required from the source-side path towards Fabric                                       |
| **Offset tracking**       | Managed by Fabric                                                                               |
| **Data movement**         | Full physical copy into OneLake                                                                 |

### Push-Based Sources

| Source                                                                                         | Mechanism               | Chapter |
| ---------------------------------------------------------------------------------------------- | ----------------------- | ------: |
| **Azure SQL Database**                                                                         | Change feed             |      11 |
| **Azure SQL Managed Instance** with **Always-up-to-date** or **SQL Server 2025** update policy | Change feed             |      12 |
| **SQL Server 2025**                                                                            | Change feed + Azure Arc |      23 |
| Azure Cosmos DB                                                                                | Continuous Backup       |      13 |
| Fabric SQL Database                                                                            | Change feed             |      24 |

### When You See Push

You will usually see this pattern when the source platform has a dedicated mirroring integration or a source-side change capture path that can send changes out as they happen. In practice, this is the method to expect for Azure SQL Database, newer Azure SQL Managed Instance paths, and SQL Server mirroring scenarios.

Push-based mirroring is generally the fastest form of mirroring because it doesn't involve a regular poll or backoff algorithm. However, the SQL Server engine can temporarily pause mirroring if it becomes busy, which may introduce a delay.

***

## Pull or Polling-Based Mirroring

In the replication model used in this book, **pull or polling-based** mirroring means Fabric connects to the source, checks for new changes, and then ingests those changes into OneLake.

### Backoff and Polling Cadence

Fabric uses a built-in backoff algorithm for polling-based sources. The purpose of this is to reduce the impact of Mirroring on the source system by reducing how frequently the replicator polls the source after repeated empty polls or transient failures.

A typical cycle works like this:

1. Fabric polls the source for changes.
2. If changes are found, Fabric ingests them and continues with the normal cadence.
3. If no changes are found, Fabric can lengthen the polling interval.
4. If transient errors occur, Fabric backs off further before retrying.
5. When work resumes or the source becomes healthy again, Fabric returns to a shorter interval.

> The backoff process is an integral part of Mirroring, and yet it is not surfaced to the user. So it's unclear when it's happening, and this can cause concerns that Mirroring is slow or has stopped working. There is no way for the user to affect this.

[![Figure 3.3: Pull-based sequence](../assets/diagrams/chapter-03/diagram-03.png)](../assets/diagrams/chapter-03/diagram-03.excalidraw.png)
*Figure 3.3: Pull-based sequence*

### How It Works

1. Fabric stores source connectivity details in a Fabric connection.
2. The replicator queries (pulls) the source for new changes since the last successful watermark, offset, or source-native checkpoint.
3. Fabric writes the returned changes to the landing zone in OneLake.
4. Fabric processes the landing zone files into Delta tables.
5. Fabric manages retries, backoff, and offset tracking.

### Characteristics

| Attribute            | Detail                                                                                          |
| -------------------- | ----------------------------------------------------------------------------------------------- |
| **Change initiator** | Fabric replicator                                                                               |
| **Latency**          | Variable, seconds/low minutes to hours depending on polling cadence, source behaviour, and load |
| **Agent required**   | No source-side agent in the usual pattern                                                       |
| **Connectivity**     | Fabric must be able to reach the source through the supported connection path                   |
| **Offset tracking**  | Managed by Fabric                                                                               |
| **Data movement**    | Full physical copy into OneLake                                                                 |
| **Backoff process**  | Differs by source                                                                                |

### Pull or Polling-Based Sources

| Source                                                                | Mechanism                                                                                    | Chapter |
| --------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- | ------: |
| **Google BigQuery**                                                   | Google Cloud Storage initial export + BigQuery `CHANGES` TVF                                 |      15 |
| **Oracle**                                                            | Oracle log-based change capture path                                                         |      16 |
| **PostgreSQL**                                                        | Logical replication                                                                          |      17 |
| **MySQL**                                                             | Binary log (binlog) replication path                                                         |      18 |
| **SAP**                                                               | SAP Datasphere replication flow to ADLS Gen2                                                 |      19 |
| **Snowflake**                                                         | Fabric polls and reads Snowflake stream changes                                              |      21 |
| **SharePoint List**                                                   | Fabric replicates list rows on a schedule; Document Library files are read through shortcuts |      20 |
| **SQL Server 2016-2022**                                              | CDC                                                                                          |      22 |
| **Azure SQL Managed Instance** with **SQL Server 2022** update policy | CDC                                                                                          |      12 |

### Snowflake: Polling Snowflake Streams

Snowflake is an interesting mix of push and polling. The replicator polls the Snowflake warehouse for changes, and if there is a change, it is written (or pushed) directly to OneLake.

Snowflake uses a polling sequence:

* Fabric checks the Snowflake stream for available changes.
* The Fabric Replication Engine reads the change set.
* Fabric writes and processes those changes into Delta tables in OneLake.

[![Figure 3.3a: Snowflake polling sequence](../assets/diagrams/chapter-03/diagram-04.png)](../assets/diagrams/chapter-03/diagram-04.excalidraw.png)
*Figure 3.3a: Snowflake polling sequence*

Fabric controls the polling schedule, but Snowflake writes the change batch directly into the OneLake landing zone once a change is found.

### SAP: Two-Step Architecture

For SAP sources, the mirroring path is also important to understand. Fabric does not poll the operational SAP tables directly in the same way it polls a database transaction log. Instead, **SAP Datasphere replication flow** first moves the relevant data towards **ADLS Gen2**, and Fabric then mirrors through that supported path. This is why SAP is best planned as a two-step architecture.

### SharePoint List: A Hybrid Case

SharePoint List mirroring does not fit neatly into a single method. Fabric replicates SharePoint list row data into Delta tables on a scheduled basis, similar to other pull or polling-based sources. Document Library data, however, is surfaced through **OneLake shortcuts** rather than being copied, similar to metadata mirroring. Plan for both behaviours when you configure a mirrored SharePoint List: list rows arrive as replicated Delta data, while document content is read in place through the shortcut path. Chapter 20 covers the full setup.

***

## Shortcuts/Metadata Mirroring

In the replication model used in this book, **shortcuts** means Fabric synchronises metadata and creates OneLake shortcuts to data that stays in its original storage location.

This is different from both push and pull or polling methods because the main data files are not copied into the mirrored database's Delta storage layer.

[![Figure 3.4: Shortcuts sequence](../assets/diagrams/chapter-03/diagram-05.png)](../assets/diagrams/chapter-03/diagram-05.excalidraw.png)
*Figure 3.4: Shortcuts sequence*

### How It Works

1. Fabric connects to the source catalog layer.
2. Fabric discovers table definitions, storage locations, and related metadata.
3. Fabric creates OneLake shortcuts that point to the source data.
4. Queries run through Fabric, but the underlying files remain in the source location.
5. Metadata refresh keeps the Fabric view aligned with the source catalog.

### Characteristics

| Attribute            | Detail                                                                                       |
| -------------------- | -------------------------------------------------------------------------------------------- |
| **Change initiator** | Fabric metadata sync                                                                         |
| **Latency**          | Metadata refresh timing depends on the connector; data freshness depends on the source files |
| **Agent required**   | No                                                                                           |
| **Connectivity**     | Fabric must be able to read the source storage and metadata endpoints                        |
| **Offset tracking**  | Not a change-feed pattern; Fabric tracks metadata refresh state instead                      |
| **Data movement**    | None for the data files                                                                      |

### Shortcut-Based Sources

| Source               | Mechanism                                  | Chapter |
| -------------------- | ------------------------------------------ | ------- |
| **Azure Databricks** | Unity Catalog metadata + OneLake shortcuts | 14      |
| **Dremio**           | Dremio metadata + OneLake shortcuts        | 25      |

Dremio support is currently in **Public Preview**.

### Fabric SQL Database Note

**Fabric SQL Database** is a special case inside Fabric. The mirrored database experience is auto-configured from metadata within the service, so there is no separate external source copy to plan in the same way as the other connectors.

***

## How the Three Methods Compare

[![Figure 3.5: Comparison](../assets/diagrams/chapter-03/diagram-06.png)](../assets/diagrams/chapter-03/diagram-06.excalidraw.png)
*Figure 3.5: Comparison*

| Attribute                          | Push-based                           | Pull or polling-based             | Shortcuts                                        |
| ---------------------------------- | ------------------------------------ | --------------------------------- | ------------------------------------------------ |
| **Initiator**                      | Source-side mechanism                | Fabric replicator                 | Fabric metadata sync                             |
| **Data copied into OneLake**       | Yes                                  | Yes                               | No                                               |
| **Operational rhythm**             | Change-driven                        | Polling-driven                    | Metadata refresh-driven                          |
| **Source dependency**              | Source-side integration required     | Source reachable to Fabric        | Source catalog and storage reachable to Fabric   |
| **Typical query path after setup** | Local Delta tables                   | Local Delta tables                | In-place reads through shortcut                  |
| **Primary replication concern**    | Connectivity and source capture path | Polling cadence, lag, and retries | Query path, storage access, and metadata refresh |

***

## Complete Source-to-Method Reference

The table below brings the source list together using the replication model in this chapter.

| Source                                                                                         | Method in this book                                  | Data moved to OneLake? | Main mechanism                                                           | Chapter |
| ---------------------------------------------------------------------------------------------- | ---------------------------------------------------- | ---------------------: | ------------------------------------------------------------------------ | ------- |
| **Azure SQL Database**                                                                         | Push-based                                           |                    Yes | Fabric mirroring change feed                                             | 11      |
| **Azure SQL Managed Instance** with **Always-up-to-date** or **SQL Server 2025** update policy | Push-based                                           |                    Yes | Fabric mirroring change feed                                             | 12      |
| **Azure SQL Managed Instance** with **SQL Server 2022** update policy                          | Pull or polling-based                                |                    Yes | SQL Server CDC via data gateway                                          | 12      |
| **Azure Cosmos DB**                                                                            | Push-based                                           |                    Yes | Continuous Backup                                                        | 13      |
| **Azure Databricks**                                                                           | Shortcuts                                            |                     No | Unity Catalog metadata + OneLake shortcuts                               | 14      |
| **Google BigQuery**                                                                            | Pull or polling-based                                |                    Yes | Google Cloud Storage initial export + BigQuery `CHANGES` TVF             | 15      |
| **Oracle**                                                                                     | Pull or polling-based                                |                    Yes | Oracle log-based change capture path                                     | 16      |
| **PostgreSQL**                                                                                 | Pull or polling-based                                |                    Yes | Logical replication                                                      | 17      |
| **MySQL**                                                                                      | Pull or polling-based                                |                    Yes | Binary log (binlog) replication path                                     | 18      |
| **SAP**                                                                                        | Pull or polling-based                                |                    Yes | SAP Datasphere replication flow to ADLS Gen2                             | 19      |
| **Snowflake**                                                                                  | Pull or polling-based                                |                    Yes | Fabric polls and creates Snowflake Streams                               | 21      |
| **SharePoint List**                                                                            | Hybrid: pull-based for rows, shortcuts for documents |                Partial | Delta tables for list rows; OneLake shortcuts for Document Library files | 20      |
| **SQL Server 2016–2022**                                                                       | Pull or polling-based                                |                    Yes | SQL Server CDC via data gateway                                          | 22      |
| **SQL Server 2025**                                                                            | Push-based                                           |                    Yes | Fabric mirroring change feed + Azure Arc + data gateway                  | 23      |
| **Dremio**                                                                                     | Shortcuts                                            |                     No | Dremio metadata + OneLake shortcuts                                      | 25      |
| **Fabric SQL Database**                                                                        | Push-based                                           |         Yes, automatic | Auto-configured inside Fabric                                            | 24      |

[![Figure 3.6: Source groupings](../assets/diagrams/chapter-03/diagram-07.png)](../assets/diagrams/chapter-03/diagram-07.excalidraw.png)
*Figure 3.6: Source groupings*

***

## Choosing the Right Method

You do not choose the method directly, but you do need to plan around it.

* **Connectivity:** Push-based sources need a supported source-side path towards Fabric. Pull or polling-based sources need Fabric to reach the source. Shortcuts need Fabric to reach the source catalog and storage path at query time.
* **Latency expectations:** Push-based sources usually have the lowest end-to-end lag. Pull or polling-based sources depend on polling cadence and source response. Shortcut freshness depends on metadata refresh and the state of the source files.
* **Storage impact:** Push-based and pull or polling-based methods create a physical copy in OneLake. Shortcuts do not.
* **Operations:** Push-based issues often centre on source integration and connectivity. Pull or polling-based issues often centre on polling, throttling, and retries. Shortcut issues often centre on metadata alignment and query access to source storage.

***

## Summary

This chapter uses a simple replication model with three methods: push-based, pull or polling-based, and shortcuts. These are book terms, not an official Microsoft taxonomy. The model helps you predict how each source behaves, what connectivity it needs, whether data is copied into OneLake, and what sort of lag you should expect. The detailed source chapters begin with Azure SQL Database in Chapter 11 and continue through Fabric SQL Database in Chapter 24.

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 2: Types of Mirroring in Fabric](chapter-02.md) | **Next:** [Chapter 4: The Anatomy of a Mirrored Database](chapter-04.md)
