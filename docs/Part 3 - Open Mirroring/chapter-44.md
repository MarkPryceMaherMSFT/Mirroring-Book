# Chapter 44: Synapse Dedicated SQL Pool Open Mirroring

> **Part 3: Open Mirroring**
>
> **Purpose:** Use the public Synapse implementation to understand snapshot/diff extraction, append-only watermarks and the separate Open Mirroring publication path.

**Part index:** [Chapters in Part 3](readme.md)

---

## Project and Customer Use

[fabric-mirroring-synapse](https://github.com/MarkPryceMaherMSFT/fabric-mirroring-synapse) contains T-SQL extraction procedures, PowerShell orchestration and an optional C# WPF desktop application. **The author confirms that this is an open-source implementation and customers may use it.** It is included as a dedicated solution chapter, not relegated to the blog-only list.

This chapter examines revision [`f9ce29f`](https://github.com/MarkPryceMaherMSFT/fabric-mirroring-synapse/tree/f9ce29f0e473a49e40dfc85c9eb2c349513b2386). No named licence file was present in that inspected tree. The author's customer-use confirmation is therefore recorded explicitly; this book does not invent an MIT or Apache licence or additional redistribution terms.

The solution offers two destinations: an Open Mirrored Database and a Fabric Warehouse. Only the first uses the Open Mirroring landing-zone protocol. Warehouse `COPY INTO` and generated DML are a separate implementation, not another name for mirroring.

## Architecture and Modes

```text
Synapse dedicated SQL pool
    -> mirror control tables and T-SQL extraction
    -> CETAS Parquet in ADLS Gen2 staging
    -> PowerShell destination synchronization
    -> Open Mirroring landing zone
    -> Fabric-managed Delta tables
```

The control-table design selects source tables, keys, modes and frequency. A FULL table uses a versioned shadow copy as the last-observed baseline. Subsequent cycles compare the current source with that baseline using set differences, classify changes by key and export change files. A HIGHWATERMARK table instead queries beyond a saved watermark and does not need a shadow comparison.

| Mode | Appropriate use | Important limitation |
|---|---|---|
| FULL with a stable key | Tables requiring detection of inserts, updates and deletes between observations | Snapshot/diff is not a transaction log; intermediate changes can be unobserved |
| HIGHWATERMARK | Genuinely append-only data with a suitable increasing extraction boundary | No update/delete detection for rows behind the watermark; late commits and ties require care |
| Keyless | Restricted set-based ingestion | Cannot reliably identify row updates/deletes; identical duplicate rows are collapsed by set difference |

The SQL engine runs on the dedicated pool. Even if only changed rows are exported, comparison can still scan substantial source and shadow data. Measure pool resource use rather than equating small output with cheap extraction.

## A Headless Setup Walkthrough

Use the project's [headless guide](https://github.com/MarkPryceMaherMSFT/fabric-mirroring-synapse/blob/f9ce29f0e473a49e40dfc85c9eb2c349513b2386/docs/HEADLESS.md) and [configuration template](https://github.com/MarkPryceMaherMSFT/fabric-mirroring-synapse/blob/f9ce29f0e473a49e40dfc85c9eb2c349513b2386/config/mirroring.config.template.json). PowerShell 7 is recommended; the staging layer requires Azure CLI, and deployment uses `sqlcmd`. Prepare source SQL permissions, external storage objects, ADLS connectivity and a Fabric publishing identity.

Work from a local checkout of the project, not from the book directory:

```powershell
Copy-Item .\config\mirroring.config.template.json .\config\mirroring.config.json
```

Fill in source SQL and ADLS staging configuration. Set `destination.type` to `MirroredDatabase`, provide the real landing-zone URL and publishing credentials, and deliberately enable destination synchronization after the source-only lab succeeds. Start with the template's COPY mode rather than assuming the optional shortcut mode works for your tenant topology.

The repository excludes the real configuration from Git. Its optional secret-protection script uses Windows DPAPI tied to the account/machine; a scheduled-task identity must be able to read its own protected configuration. Do not copy a protected configuration to a different host and assume it will decrypt.

```powershell
.\ps\Protect-MirrorConfigSecrets.ps1 -ConfigPath .\config\mirroring.config.json
.\deploy.ps1 -ConfigPath .\config\mirroring.config.json
```

Review deployment SQL before running it against an existing pool. Register one representative keyed table, substituting a real test schema/table:

```powershell
.\ps\Manage-Mirroring.ps1 -Action AddTable `
    -SchemaName dbo -TableName Customer -KeyColumn CustomerId
.\ps\Manage-Mirroring.ps1 -Action ListTables
```

Read that table's `ControlID` from the result, then scope the run to it:

```powershell
.\ps\Manage-Mirroring.ps1 -Action Run -ControlID 8 -SkipSync
.\ps\Manage-Mirroring.ps1 -Action Run -ControlID 8
.\ps\Manage-Mirroring.ps1 -Action FabricStatus
```

`8` is an example, not a predefined table identifier. An unscoped `Run` can process every due registered table. `-SkipSync` exercises source extraction/staging only; it does not prove that Fabric received anything.

## Row Operations and Source Behaviour

The current SQL uses marker `4` for non-delete rows and `2` for physical deletes; optional soft deletes publish an upsert with `__softDelete__ = 1`. The row marker is placed last, after any additional fields. The [watermark procedure](https://github.com/MarkPryceMaherMSFT/fabric-mirroring-synapse/blob/f9ce29f0e473a49e40dfc85c9eb2c349513b2386/sql/14_ReplicateWatermark_Procedure.sql) emits `4`, despite an older paragraph in the README describing `0`.

Upsert semantics reduce duplicate-key insertion on repeat delivery, but do not make out-of-order replay harmless. An old upsert applied after a newer change can still restore stale values.

Initial landing data and the shadow baseline are separate extraction work. Under concurrent source writes, establish what consistency boundary they represent and test changes between those operations. The same applies when refreshing the shadow after a diff. Do not describe periodic state comparison as lossless transaction-level CDC.

## Publication, Ordering and Recovery

The [PowerShell module](https://github.com/MarkPryceMaherMSFT/fabric-mirroring-synapse/blob/f9ce29f0e473a49e40dfc85c9eb2c349513b2386/ps/MirroringLib.psm1) writes metadata with `LastUpdateTimeFileDetection` for nonsequential CETAS output. Its `Send-OneLakeFileBytes` helper uses create/append/flush against the supplied path. That is not yet the complete temporary-file/atomic-rename/immutable-publication pattern from [Chapter 34](chapter-34.md).

Separate three progress points: source/shadow extraction, staged-file synchronization, and Fabric ingestion. `LastSyncedUtc` is a publisher-side file watermark, not a source transaction boundary or a Fabric ingestion acknowledgement. Timestamp detection also does not guarantee logical source ordering across late files or overlapping cycles.

Keep a durable staged-file manifest and batch assignment, publish complete files atomically, reconcile ambiguous responses and only advance delivery progress for confirmed publications. Do not remove staging files needed for recovery because source extraction has moved forward.

The repository provides reset and schema-change handling, but automatic immediate recreation must be reconciled with current Fabric guidance: suspend publication, delete the table folder, wait for disappearance, recreate metadata and a complete load, then wait for the table to reappear before resuming changes. Reset is destructive, not a generic retry.

## Status, Maturity and Costs

The project exposes `mirror.MirrorControl`, `mirror.MirrorLog`, batch logs and `FabricStatus`. Compare source-side status with Fabric's database/table APIs and actual destination values. A successful CETAS or a healthy control row is not proof of end-to-end delivery.

At the inspected revision, the module header still says the OneLake REST helpers were not field-tested, while other repository sections describe specific observations and Warehouse testing. Record this evidence mismatch rather than claiming a complete Open Mirroring acceptance run. Warehouse success does not prove the mirrored-database branch.

Test keyed insert/update/delete, a soft delete, concurrent source changes during snapshot/diff, lost upload responses, source schema changes and staged-file recovery. Inspect source pool overhead and retain sufficient staging history. Dedicated-pool compute, ADLS storage/operations, orchestration and Fabric analytics remain costs even though customers can use the implementation.

**Further reading:** [project README](https://github.com/MarkPryceMaherMSFT/fabric-mirroring-synapse#readme), [desktop quick start](https://github.com/MarkPryceMaherMSFT/fabric-mirroring-synapse/blob/main/QUICKSTART.md), and [shared recovery guidance](chapter-35.md).

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 43: PostgreSQL Polling with impulse_sync](chapter-43.md) | **Next:** [Chapter 45: File Publishing and Open Mirroring Test Tools](chapter-45.md)
