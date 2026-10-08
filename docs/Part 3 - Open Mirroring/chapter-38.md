# Chapter 38: Toolbox Notebook Solutions - Excel, SharePoint, MySQL, and Snowflake

> **Part 3: Open Mirroring**
>
> **Purpose:** Learn from executable source-specific notebooks without confusing a scheduled demonstration with a complete replication service.

**Part index:** [Chapters in Part 3](readme.md)

---

## Why Group These Examples?

The [Toolbox sample collection](https://github.com/microsoft/fabric-toolbox/tree/b0183fb1841367fd0eaa4aae28949f7911ff4f05/samples/open-mirroring) contains several separate Python notebooks. They share the extraction-to-Parquet-to-OneLake pattern, but their source mechanisms differ considerably. Read them alongside the [SDK chapter](chapter-36.md), not as evidence that the SDK itself knows how to capture changes.

The inspected revision is `b0183fb`, and the [MIT licence](https://github.com/microsoft/fabric-toolbox/blob/b0183fb1841367fd0eaa4aae28949f7911ff4f05/LICENSE) covers the repository source. The collection labels the examples proof-of-concept code, not production-ready software. Source-service licences and Fabric notebook compute remain separate costs.

## Source Map

| Notebook | Real source code | Capture pattern |
|---|---|---|
| Excel Mirroring | [excelmirroring.ipynb](https://github.com/microsoft/fabric-toolbox/blob/b0183fb1841367fd0eaa4aae28949f7911ff4f05/samples/open-mirroring/Excel%20Mirroring/excelmirroring.ipynb) | Scan a OneLake folder and publish workbook sheets |
| SharePoint Excel | [sharepoint-excel-mirroring.ipynb](https://github.com/microsoft/fabric-toolbox/blob/b0183fb1841367fd0eaa4aae28949f7911ff4f05/samples/open-mirroring/SharepointExcelMirroring/sharepoint-excel-mirroring.ipynb) | Graph download followed by workbook conversion |
| SharePoint Lists | [sharepoint-list-mirroring.ipynb](https://github.com/microsoft/fabric-toolbox/blob/b0183fb1841367fd0eaa4aae28949f7911ff4f05/samples/open-mirroring/SharepointListMirroring/sharepoint-list-mirroring.ipynb) | Graph list response converted to a table |
| MySQL | [mysqlmirroring.ipynb](https://github.com/microsoft/fabric-toolbox/blob/b0183fb1841367fd0eaa4aae28949f7911ff4f05/samples/open-mirroring/mysql%20Mirroring/mysqlmirroring.ipynb) | Initial snapshot plus source triggers and a change table |
| Snowflake | [snowflakeMirroring.ipynb](https://github.com/microsoft/fabric-toolbox/blob/b0183fb1841367fd0eaa4aae28949f7911ff4f05/samples/open-mirroring/Snowflake%20Mirroring/snowflakeMirroring.ipynb) | Snapshot plus Snowflake stream extraction |

These are custom publishers. They are not the native SharePoint, MySQL or Snowflake connectors described in Part 2.

## Getting Started

Import the selected notebook into a test Fabric environment, install its stated dependencies, and identify every configuration and credential cell before execution. Grant source access separately from OneLake write access. Confirm that the network path exists from the notebook runtime; a working desktop connection does not prove that.

Create an empty mirrored database and select one small keyed source object. Read every reset/setup cell before running the notebook: some delete destination tables or recreate source capture objects. Run initialization once, then schedule only the intended incremental cells after making their recovery safe.

Use [Chapter 34](chapter-34.md) to inspect metadata, row-marker placement and file names. Embedded helper classes can differ from the standalone SDK. Do not assume that a method added to one notebook exists in `OpenMirroringPythonSDK`.

## Excel and SharePoint: Current State Is Not a Change Stream

The workbook notebooks convert current sheet contents to Parquet, generate positional row identities and publish update/upsert-like rows. Reordering the workbook changes that identity, while removing rows does not automatically generate deletes. The `clean` argument in the Excel helper is compared with the string `"true"`; it controls a destructive remove/recreate path rather than an ordinary incremental refresh.

The SharePoint Excel notebook adds a Graph download step. The inspected function processes a returned `children` page; pagination, deleted-file reconciliation and a durable download cursor are additional work. Similarly, the list notebook converts a returned list response rather than implementing a complete paginated delta protocol.

For a real synchronization service, keep stable source keys, fetch all pages, retain the last successfully published snapshot and publish missing-key deletes. The [file pattern in Chapter 33](chapter-33.md#use-case-4-excelcsv-mirroring) explains the division of responsibilities.

## MySQL: Trigger Capture and Delivery Acknowledgement

`setup_cdc_for_table` creates the source change table and insert/update/delete triggers. It can drop and recreate those objects, so it is not a harmless restart function. Inspect the generated SQL with the source owner before using it.

`export_cdc_to_parquet` selects unmoved changes, writes a local file, then marks currently unmoved rows as moved before a later upload cell runs. This separates acknowledgement from delivery. New changes arriving between selection and the broad update can also be acknowledged without belonging to that export.

A hardened design captures a bounded, ordered set of change identifiers, journals its payload, publishes it, and acknowledges exactly that set. It also establishes a snapshot/trigger boundary and handles primary-key changes as removal of the old identity plus publication of the new identity. A retry loop alone cannot fix an incorrectly advanced source checkpoint.

## Snowflake: Stream Progress Is a Source Checkpoint

The Snowflake notebook identifies object types, creates streams, extracts snapshots and queries stream changes. In the inspected incremental cell, `create_stream_if_supported` uses `CREATE OR REPLACE STREAM` after reading changes and before uploading them.

Replacing a stream and publishing a OneLake file are not one transaction. Preserve the extracted changes and protect the source boundary before acknowledging or resetting the stream. Sorting by row ID and action is not a general guarantee of transaction order.

The notebook also defines its helper class after earlier cells that use it, so a clean top-to-bottom execution needs attention. This is precisely why source inspection is more useful than simply listing a notebook link.

## Lessons and Acceptance Exercises

Interrupt each notebook between export and upload, and again between upload and progress persistence. Test source changes during the initial scan, pagination, deletions, key changes and a schema change. Reconcile the resulting rows, not just the presence of a file.

These examples make source integration approachable. Their strongest lesson is that source capture, publication and acknowledgement are three separate steps. Use the [shared recovery guidance](chapter-35.md) and [acceptance exercises](chapter-46.md#acceptance-exercises) before scheduling them unattended.

**Related article:** [Mirroring Excel into Fabric with Open Mirroring (2nd try)](https://medium.com/@sqltidy/mirroring-excel-into-fabric-with-open-mirroring-2nd-try-83690d950cf6) has matching Toolbox source and belongs here, not in the blog-only table.

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 37: GenericMirroring](chapter-37.md) | **Next:** [Chapter 39: MariaDB Through MaxScale and Kafka](chapter-39.md)
