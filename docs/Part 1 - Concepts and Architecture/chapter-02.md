# Chapter 2: Types of Mirroring in Fabric

> **Part 1: Concepts and Architecture**
>
> **Purpose:** This chapter explains the three official mirroring types in Fabric and shows which sources belong to each one so you can choose the right pattern for your source.

***

## Overview

Chapter 1 introduced mirroring as a single idea: a managed replica of source data available in Fabric. In practice, Fabric implements that idea through three distinct **mirroring types**, and the type determines what Fabric creates in your workspace, whether your data is physically copied into OneLake, and who is responsible for keeping it current.

This chapter walks through each type in turn: what is replicated, what Fabric creates, its key characteristics, and which sources use it today. Use it to decide which type applies to your source before you plan connectivity, security, or operations for a mirror. Chapter 3 then covers the transport methods (push, pull or polling, and shortcuts) that sit underneath these types.

Fabric groups mirroring into three types:

* **Database mirroring** replicates data into OneLake. (Moves data)
* **Metadata mirroring** syncs metadata and points Fabric to data that stays in the source. (Doesn't move data)
* **Open mirroring** lets users build their own mirroring solution. (Moves data)

These types are different from the transport methods covered in Chapter 3. The type tells you what Fabric creates and where the data lives.

## Database Mirroring

Database mirroring is the standard pattern for supported source databases and platforms. Fabric reads source data, writes it into OneLake, and materialises Delta tables that can be queried from Fabric workloads.

**What is replicated:**

* Table and schema definitions
* Source row data
* Ongoing inserts, updates, and deletes, subject to source support

**What is created in Fabric:**

* A **Mirrored Database** item
* A landing zone in OneLake
* Delta tables in OneLake
* An auto-provisioned **SQL analytics endpoint**

**Key characteristics:**

* Data is physically copied into OneLake.
* Fabric manages replication for supported sources.
* Schema changes are handled according to source-specific rules.
* Query performance and behaviour depend on the Fabric workload you use on top of the mirrored tables.
* Delta tables are v-order enabled

**Supported sources for database mirroring:**

* Azure SQL Database
* Azure SQL Managed Instance
* Azure Cosmos DB
* Azure Database for PostgreSQL
* Azure Database for MySQL *(Public Preview)*
* Google BigQuery *(Public Preview)*
* Oracle
* SAP
* SharePoint List *(Public Preview)*
* Snowflake
* SQL Server 2016–2022
* SQL Server 2025
* Fabric SQL Database *(auto-configured)*

SharePoint List is classified as database mirroring, but it uses a hybrid mechanism: Document Library data is surfaced through OneLake shortcuts, and list row data is replicated into Delta tables. Chapter 20 covers this in detail.

Snowflake also supports **view replication** as a separate paid extended capability. That capability is covered in Chapter 9.

> A key point of Database Mirroring is that control is handed off to Fabric, so the user doesn't control when the replication happens, or add any transformations into the mirroring process. This does not suit everyone, so if more control is needed there are plenty of other ETL, ELT, or near real-time streaming solutions in Fabric.

## Metadata Mirroring

Metadata mirroring does not copy table data into OneLake. Instead, Fabric syncs metadata from the source catalogue and exposes the data through **OneLake shortcuts**. The data remains in the source system.

**What is replicated:**

* Catalogues, schemas, and table metadata
* Storage references used to access the data in place

**What is created in Fabric:**

* A mirrored database item
* OneLake shortcuts that point to the source data
* Registered tables visible from SQL and Spark

**Key characteristics:**

* Data stays in the source platform.
* Fabric keeps metadata in sync rather than replicating rows.
* Query execution reads through the shortcut path to the source data location.
* This type is useful when the source already stores data in an open table format and Fabric only needs to surface it.

**Supported sources for metadata mirroring:**

* Azure Databricks (Unity Catalog) *(GA)*
* Dremio *(Public Preview)*

For Dremio, Fabric uses **credential vending** or a personal access token to access the underlying Iceberg data through the mirrored metadata path. Chapter 25 covers Dremio catalog mirroring in detail.

> [Microsoft Dataverse direct link to Microsoft Fabric](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/azure-synapse-link-view-in-fabric) is not technically in the Mirroring family, but it works in the same way as Metadata Mirroring, so it's worth mentioning here.

Metadata Mirroring uses shortcuts, so no data is moved and there is no lag before changes appear in Fabric.

## Open Mirroring

Open mirroring is the extensibility model. A partner solution or custom application writes data and metadata files into the open mirroring landing zone, and Fabric processes them into Delta tables.

**What is replicated:**

* Any data the connector or application chooses to publish in the required format

**What is created in Fabric:**

* An **Open Mirrored Database** item
* Delta tables built from the supplied files
* An auto-provisioned SQL analytics endpoint

**Key characteristics:**

* Open Mirroring is Generally Available.
* Fabric provides the target format and processing model.
* The connector or application is responsible for extraction logic, source-specific change tracking, and operational behaviour.
* Individual partner solutions vary in packaging, support model, and feature coverage.
* Extremely flexible

> **Example sources:** Open mirroring suits sources without a native connector, such as unsupported on-premises databases, SaaS applications, or custom application event streams. Microsoft publishes sample connectors and partner examples at **aka.ms/FabricToolbox**.

## Comparison Matrix

| Attribute                           | Database mirroring                                     | Metadata mirroring              | Open mirroring                              |
| ----------------------------------- | ------------------------------------------------------ | ------------------------------- | ------------------------------------------- |
| **Data copied into OneLake**        | Yes                                                    | No                              | Yes                                         |
| **Fabric-managed source connector** | Yes                                                    | Yes                             | No                                          |
| **Primary output**                  | Delta tables in OneLake                                | Metadata plus OneLake shortcuts | Delta tables in OneLake                     |
| **SQL analytics endpoint**          | Yes                                                    | Yes                             | Yes                                         |
| **Spark access**                    | Yes                                                    | Yes                             | Yes                                         |
| **Source coverage**                 | Fixed supported source list                            | Fixed supported source list     | Any source with a compatible implementation |
| **Operational responsibility**      | Mostly Fabric                                          | Shared with source platform     | Application owner                           |
| **Typical examples**                | Azure SQL Database, Snowflake, Oracle, SharePoint List | Azure Databricks, Dremio        | Custom applications, ISV connectors         |

## Selection Decision Framework

Use this decision path to choose the right type.

[![chapter-02 diagram 1](../assets/diagrams/chapter-02/diagram-01.png)](../assets/diagrams/chapter-02/diagram-01.excalidraw.png)

*Figure 2.1: Choosing a mirroring type*

**Additional considerations:**

* Choose **database mirroring** when you need the data physically present in OneLake.
* Choose **metadata mirroring** when the source already stores the data externally and you only need Fabric to surface it through synced metadata and shortcuts.
* Choose **open mirroring** when the source is not natively supported but you can use or build a connector that emits the required files.
* Check release status before committing to a source. Preview connectors can change more quickly than Generally Available ones.
* Separate the mirroring type decision from the transport method decision. Chapter 3 covers push, pull or polling, and shortcut-based access.

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 1: Introduction to Fabric Mirroring](chapter-01.md) | **Next:** [Chapter 3: Methods of Mirroring: Push, Pull or Polling, and Shortcuts](chapter-03.md)
