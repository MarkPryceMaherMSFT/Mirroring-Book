# Chapter 37: GenericMirroring - A Multi-Source C# Publisher

> **Part 3: Open Mirroring**
>
> **Purpose:** Understand the multi-source Toolbox proof of concept and identify what must change before operating it as a reliable connector.

**Part index:** [Chapters in Part 3](readme.md)

---

## Project and Scope

[GenericMirroring](https://github.com/microsoft/fabric-toolbox/tree/main/samples/open-mirroring/GenericMirroring) is a .NET 8 application with adapters for SQL Server Change Tracking, local Excel workbooks, CSV files, Access databases and SharePoint Lists. It is useful when the reader wants to inspect a small source-adapter framework rather than start with a blank project.

The [collection explicitly calls these proof-of-concept samples](https://github.com/microsoft/fabric-toolbox/blob/b0183fb1841367fd0eaa4aae28949f7911ff4f05/samples/open-mirroring/README.md), not production connectors. This chapter uses revision `b0183fb`. The publisher code is covered by the [Toolbox MIT licence](https://github.com/microsoft/fabric-toolbox/blob/b0183fb1841367fd0eaa4aae28949f7911ff4f05/LICENSE).

**Excel dependency caveat:** the [project references EPPlus 7.5.3](https://github.com/microsoft/fabric-toolbox/blob/b0183fb1841367fd0eaa4aae28949f7911ff4f05/samples/open-mirroring/GenericMirroring/GenericMirroring.csproj). That version has [Polyform Noncommercial/commercial licensing](https://github.com/EPPlusSoftware/EPPlus/blob/213a22d8c0f0fe98086b0240a1f8a12d56109483/license.md). The sample's noncommercial setting does not grant commercial-use rights. Do not describe the complete Excel branch as unrestricted free software for businesses.

## Architecture and Setup

```text
mirrorconfig.json
    -> source polling or file watcher
    -> DataTable and Parquet conversion
    -> local staging
    -> enabled OneLake destinations
```

Read [Program.cs](https://github.com/microsoft/fabric-toolbox/blob/b0183fb1841367fd0eaa4aae28949f7911ff4f05/samples/open-mirroring/GenericMirroring/Program.cs), the [configuration](https://github.com/microsoft/fabric-toolbox/blob/b0183fb1841367fd0eaa4aae28949f7911ff4f05/samples/open-mirroring/GenericMirroring/mirrorconfig.json), and [Upload.cs](https://github.com/microsoft/fabric-toolbox/blob/b0183fb1841367fd0eaa4aae28949f7911ff4f05/samples/open-mirroring/GenericMirroring/Upload.cs) together. Configuration alone does not describe the complete execution path.

Start on a Windows test host with .NET 8, the required source drivers, and source/OneLake connectivity. Access uses the ACE OLE DB provider; the AzCopy path also expects its executable and PowerShell. Configure one source table and one destination first. Set enable flags explicitly: parts of the program gate execution on settings missing from the checked-in example.

Prepare the Fabric item and identity using [Chapter 31](chapter-31.md), protect real credentials, and keep the configuration out of source control. Review SQL statements before enabling Change Tracking on a source. File watchers are not a substitute for an initial directory scan: prove that pre-existing files are handled as well as newly changed ones.

## What Each Adapter Actually Does

| Adapter | Change detection | Important boundary |
|---|---|---|
| SQL Server | Initial extraction and `CHANGETABLE(CHANGES...)` polling | Change Tracking is not transaction-log CDC; source retention and a consistent version boundary matter |
| Excel | Re-read worksheets and compare cached contents | Positional `_id_`; current rows are sent with update/upsert-like semantics, but removed rows are not explicit deletes |
| CSV | Read the full file and compare cached contents | Splits on commas rather than using a quoted-field-aware parser |
| Access | Read tables/views through OLE DB | Positional identity and whole-result extraction, not a source change stream |
| SharePoint Lists | Graph request and returned field extraction | Inspected extraction does not traverse continuation pages or a delta cursor |

The adapter implementations are under [sources](https://github.com/microsoft/fabric-toolbox/tree/b0183fb1841367fd0eaa4aae28949f7911ff4f05/samples/open-mirroring/GenericMirroring/sources). Test nulls, dates, decimals and large integers through [Parquet.cs](https://github.com/microsoft/fabric-toolbox/blob/b0183fb1841367fd0eaa4aae28949f7911ff4f05/samples/open-mirroring/GenericMirroring/Parquet.cs), not just string-only demonstration data.

## Checkpoints, Publication and Restarts

In the inspected SQL loop, a database-level high watermark is advanced around individual table extracts; extraction and current-version lookup are separate operations. That is not a durable acknowledgement that every table through that version has reached OneLake. A safe adaptation needs consistent source bounds and progress scoped to the work actually delivered.

The [upload implementation](https://github.com/microsoft/fabric-toolbox/blob/b0183fb1841367fd0eaa4aae28949f7911ff4f05/samples/open-mirroring/GenericMirroring/Upload.cs) also needs hardening: the direct-storage path writes final filenames with overwrite enabled, while the AzCopy path starts a process without awaiting a verified exit result. Incremental callers can advance state without awaiting successful remote publication.

These are findings from source inspection, not claims of observed customer incidents. They explain why the project's proof-of-concept label matters. Adapt the immutable, atomic publication and durable assignment design from Chapters [31](chapter-31.md) and [34](chapter-34.md) before attaching a reliable source checkpoint.

For file-based adapters, replace row-position keys with business keys. Compare the new accepted snapshot with the previous accepted snapshot and emit explicit deletes for missing keys. An in-memory cache cannot preserve that comparison across process loss. Also ensure the row marker is last: some sample layouts precede later-added identity columns and differ from the current contract.

## A Useful Evaluation Exercise

Use two SQL tables, not one: change both during extraction and confirm neither loses changes when progress advances. Then restart during upload and inspect both the source version and published file assignment. For files, reorder rows, remove a row, empty the workbook, and introduce a quoted CSV field containing a comma.

Keep source checks, local export success and Fabric ingestion status separate. Broadcast to several destinations requires a separate acknowledgement for each destination; a single global watermark is not sufficient evidence that all copies are current.

**Not implemented here:** the README's Synapse Gen2, BigQuery, Redshift and ODBC TODO entries do not constitute working adapters. Read the separate [FabricBQSync](chapter-40.md) and [Synapse](chapter-44.md) chapters. The archived SQL Server and Excel projects are predecessors, not additional current connectors.

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 36: The Microsoft Open Mirroring Python SDK](chapter-36.md) | **Next:** [Chapter 38: Toolbox Notebook Solutions](chapter-38.md)
