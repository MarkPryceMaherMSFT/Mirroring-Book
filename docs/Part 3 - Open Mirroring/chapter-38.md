# Chapter 38: BigQuery with FabricBQSync

> **Part 3: Open Mirroring**
>
> **Purpose:** Evaluate a configurable BigQuery accelerator that can publish through Open Mirroring, separately from Fabric's native BigQuery connector.

**Part index:** [Chapters in Part 3](readme.md)

---

## Project and Fit

[microsoft/FabricBQSync](https://github.com/microsoft/FabricBQSync) is a Microsoft-published accelerator with [MIT-licensed source](https://github.com/microsoft/FabricBQSync/blob/a8bf1f1bf26de9fdb2818322ec41d182ce9b511a/LICENSE). It supports multiple destination modes. For this chapter, explicitly choose **`MIRRORED_DATABASE`**; a successful Lakehouse synchronization is not evidence that Open Mirroring was used.

The inspected revision is `a8bf1f1`, with release log version 2.2.0 and mirrored-database support recorded in 2.1.0. Review the [release log](https://github.com/microsoft/FabricBQSync/blob/a8bf1f1bf26de9fdb2818322ec41d182ce9b511a/Docs/ReleaseLog.md) and pin the version used in your lab. Repository ownership is not a product-support commitment; the inspected support file does not establish one.

Choose this project when you want its configurable extraction and scheduling machinery and are prepared to own its Fabric Spark execution. Compare that ownership and cost with the [native BigQuery guide](../Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-16.md).

## Architecture

```text
BigQuery extraction strategy
    -> Fabric Spark DataFrame
    -> row-operation and type conversion
    -> per-table scratch Parquet
    -> numbered landing-zone files
    -> Fabric ingestion
```

Trace [Loader.py](https://github.com/microsoft/FabricBQSync/blob/a8bf1f1bf26de9fdb2818322ec41d182ce9b511a/Packages/FabricSync/FabricSync/BQ/Loader.py), [Mirror.py](https://github.com/microsoft/FabricBQSync/blob/a8bf1f1bf26de9fdb2818322ec41d182ce9b511a/Packages/FabricSync/FabricSync/BQ/Mirror.py), and [FileSystem.py](https://github.com/microsoft/FabricBQSync/blob/a8bf1f1bf26de9fdb2818322ec41d182ce9b511a/Packages/FabricSync/FabricSync/BQ/FileSystem.py) together. The source query, Spark output partitioning and publication sequence are separate parts of the implementation.

## Setup Walkthrough

Follow the project's [installation guide](https://github.com/microsoft/FabricBQSync/blob/a8bf1f1bf26de9fdb2818322ec41d182ce9b511a/Docs/Installation.md). Import the installer notebook, attach the required Lakehouse for supporting metadata, configure GCP service-account access and select the project/dataset. Select the mirrored-database target and enable the schema configuration required by that path.

Keep credentials in protected configuration. Record the installed package version because automatic upgrades can change the code beneath a scheduled run. Begin with a small keyed table and an explicit extraction strategy. Prepare the destination and permissions using [Chapter 29](chapter-29.md).

Inspect the initial file, table key and destination row values before enabling broad table discovery or scheduling. Initialization/overwrite paths can drop a mirrored table folder and reseed it; they are not harmless incremental operations.

## Changes, Keys and Schema

The query builder includes BigQuery `CHANGES`/`APPENDS` strategies and ordinary watermark predicates. These have different source prerequisites and completeness guarantees. A strict timestamp predicate does not acquire CDC semantics just because its output is sent to an Open Mirrored Database.

The mirror conversion maps keyed change operations to Fabric markers, handles initial/no-key insert semantics and converts complex values to JSON strings. Internal source CDC columns are removed. The inspected row selection puts the marker before data columns, which differs from the current documented final-column requirement; resolve that discrepancy against [Chapter 32](chapter-32.md) when preparing the version you deploy.

Also test multiple changes to one key within a window. Spark partitioning and numbered output files do not independently establish source-event order after source ordering fields have been removed.

## Progress and Partial Publication

The filesystem code discovers file progress, checks the expected next index and checks rename results before incrementing. This is useful implementation discipline, but it is not a complete crash-safe transaction spanning the source query and every published file.

[Schedule telemetry](https://github.com/microsoft/FabricBQSync/blob/a8bf1f1bf26de9fdb2818322ec41d182ce9b511a/Packages/FabricSync/FabricSync/BQ/SyncUtils.py#L723-L756) records both `max_watermark` and `mirror_file_index`. Keep them distinct from Fabric's ingestion completion.

An especially useful exercise is to fail the second file of a multi-file publication. On retry, require reconciliation of the already-published first file and its source batch. Detecting a sequence mismatch is valuable; choosing another sequence or resetting the table automatically is not a general recovery solution.

## Operational Lessons

Monitor BigQuery query volume, Spark execution, source progress, staged files, published sequence and destination freshness. Include keyed updates/deletes and change-history expiry in the evaluation, not just a large initial load.

Free Fabric replication compute does not make the extractor free. Budget BigQuery query charges, Fabric Spark, supporting storage and possible network egress. The project's value is inspectable orchestration and source-specific logic; the reader still owns its configuration, reliability and upgrades.

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 37: MariaDB Through MaxScale and Kafka](chapter-37.md) | **Next:** [Chapter 39: MongoDB Through Change Streams](chapter-39.md)
