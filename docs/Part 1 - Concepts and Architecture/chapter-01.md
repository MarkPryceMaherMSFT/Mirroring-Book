# Chapter 1: Introduction to Fabric Mirroring

> **Part 1: Concepts and Architecture**
>
> **Purpose:** This chapter explains what Fabric Mirroring is, the problem it solves, how it works, and how mirrored storage costs are calculated so you can decide whether it fits your workload.

**Part index:** [Chapters in Part 1](readme.md)

***

Fabric Mirroring makes supported operational and analytical data available to Fabric workloads. Database mirroring replicates source data into Delta tables in OneLake. Metadata mirroring exposes data in place through shortcuts, while open mirroring accepts changes supplied by a custom or partner integration.

For database mirroring, you configure a supported source and Fabric manages replication. You can then use the replicated data from SQL, Spark, Power BI, and other Fabric workloads.

> I avoid treating "near real-time" as a latency promise. The useful question is how fresh the data needs to be for your workload, and whether your chosen source and query path can meet that requirement.

## 1.1 What Is Microsoft Fabric?

Microsoft Fabric is a Software as a Service analytics platform that combines data integration, engineering, warehousing, real-time analytics, data science, and business intelligence in one product.

Fabric is built around **OneLake**, the tenant-wide storage layer used by Fabric workloads. OneLake stores data in open formats and provides a common location for lakehouses, warehouses, shortcuts, and mirrored data.

Mirroring is one of Fabric's data integration options. Its role is to make source data available for analytics, either as a managed replica or through a supported metadata and shortcut integration.

## 1.2 Release History

The [Book Update History](../history.md) records dated Fabric Mirroring milestones and documentation changes. Use the [appendix](../appendix.md) for the current source and availability matrix.

## 1.3 The Problem Mirroring Solves

Before mirroring, teams usually had to move operational data into an analytics platform by building ETL or ELT pipelines. Those pipelines often create a few recurring problems:

* Data arrives on a schedule rather than when changes happen.
* Source systems take extra read load from reporting and ad hoc analysis.
* Pipelines need ongoing maintenance for schema changes, failures, credentials, and retries.
* Different tools produce different storage layouts and metadata conventions.
* Analysts often need separate access paths for the same data.

Database mirroring addresses these issues by keeping a managed replica in Delta format in OneLake, available to multiple Fabric workloads without separate custom ingestion code for each consumer. It does not replace transformation pipelines, source administration, or monitoring.

## 1.4 Mirroring Compared with Custom Ingestion

This comparison applies to database mirroring. Metadata mirroring and open mirroring have different storage and operational responsibilities, covered in [Chapter 2](chapter-02.md).

| Feature        | Fabric Mirroring                                                                                                                                                                                                     | Traditional ETL                                                     |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| Setup          | Guided connector setup, plus source permissions and network preparation | Pipeline design, configuration, and deployment |
| Data freshness | Continuous replication with source-dependent latency | Batch or streaming, depending on the implementation |
| Maintenance    | Fabric manages replication; you manage prerequisites, credentials, and monitoring | You own the ingestion pipeline and its operation |
| Storage format | Delta Lake in OneLake                                                                                                                                                                                                | Varies by tool and destination                                      |
| Consumption    | Native access from SQL, Spark, and Power BI                                                                                                                                                                          | Often needs extra modelling or connectors                           |
| Cost           | Replication compute is free. Mirrored storage is free up to 1 TB per purchased capacity unit (an F4 capacity includes 4 TB free mirrored storage). Querying via SQL, Spark, or Power BI is charged at standard rates. | Separate ingestion compute plus destination storage and query costs |

Mirrored storage allowance is calculated at the capacity level. The free allowance scales with purchased capacity units rather than with the number of mirrored databases. Source-side charges, networking, gateway hosting, querying, and optional extended capabilities can still incur costs. See [Chapter 10](chapter-10.md) and [Cost of mirroring](https://learn.microsoft.com/en-us/fabric/mirroring/overview#cost-of-mirroring).

## 1.5 Ideal Use Cases

Mirroring works best when you need current source data in Fabric without building a separate ingestion estate.

| Use case                       | Why mirroring fits                                                           |
| ------------------------------ | ---------------------------------------------------------------------------- |
| **Operational analytics**      | Report on current business data without querying the source directly.        |
| **Cross-system analysis**      | Combine data from multiple source systems in one Fabric workspace.           |
| **Reducing read load**         | Move reporting and analytical reads away from production databases.          |
| **Access control separation**  | Let analysts work in Fabric without direct source-system access.             |
| **Historical change analysis** | Use supported change feeds or downstream history tables; a current-state replica alone is not an archive. |
| **Validation and reconciliation** | Compare the analytical copy with the source, allowing for replication lag. |

## 1.6 High-Level Architecture and Workflow

For database mirroring, the broad pattern is an initial snapshot followed by changes, staged in a landing zone and applied to Delta tables. The source integration determines who extracts and sends those changes. Metadata mirroring does not use this row-replication pipeline, and open mirroring makes the publisher responsible for extraction.

[![chapter-01 diagram 1](../assets/diagrams/chapter-01/diagram-01.png)](../assets/diagrams/chapter-01/diagram-01.excalidraw.png)

*Figure 1.1: Database mirroring architecture*

**Key components:**

* **Source system**: The database, platform, or application being mirrored.
* **Fabric Replication Engine**: The managed processing that applies incoming changes to the mirrored tables.
* **Landing zone**: The OneLake location where mirroring writes source files before they are materialised as tables.
* **Delta tables**: The queryable tables created from mirrored data in OneLake.
* **SQL analytics endpoint**: The auto-provisioned T-SQL endpoint over the mirrored tables.

For supported database connectors, Fabric manages the replication pipeline alongside the source-specific integration. The change capture mechanism and supported schema changes depend on the source.

Fabric supports three official mirroring types:

* **Database mirroring** copies source data into OneLake.
* **Metadata mirroring** syncs metadata and uses shortcuts to data that remains in the source.
* **Open mirroring** lets a custom or partner solution write data into the mirroring landing zone.

## What comes next

The next chapter separates the three mirroring types and shows which sources belong to each one. That distinction matters because setup, data movement, and operational responsibility differ by type.

**Contents:** [Table of Contents](../index.md) | **Next:** [Chapter 2: Types of Mirroring in Fabric](chapter-02.md)
