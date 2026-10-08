# Chapter 33: Common Issues and Troubleshooting

> **Part 3: Open Mirroring**
>
> **Purpose:** Use this chapter to diagnose mirroring failures by symptom and work from monitoring evidence to source, connectivity, schema, and ordering checks.

**Part index:** [Chapters in Part 3](readme.md)

---

## Overview

This chapter provides diagnostic steps for common built-in and Open Mirroring failures.

Start with [Monitor replication](https://learn.microsoft.com/en-us/fabric/mirroring/monitor) and the affected table's error. Distinguish source extraction, landing-zone publication, OneLake processing, and SQL endpoint metadata synchronisation. A healthy database-level status does not prove every table is current.

[![Figure 33.1: Troubleshooting decision tree for mirroring issues](../assets/diagrams/chapter-33/diagram-01.png)](../assets/diagrams/chapter-33/diagram-01.excalidraw.png)
*Figure 33.1: Troubleshooting decision tree for mirroring issues*

---

## File Format and Schema Errors

### Issue: "Invalid Parquet file" or "Schema mismatch"

**Symptoms**: Fabric rejects landing zone files; table does not appear in the SQL analytics endpoint.

**Causes and resolutions:**

| Cause | Resolution |
|---|---|
| Parquet file is corrupted or incomplete | Validate the complete file before atomically renaming it into place. Use the failed-file repair procedure below for an already rejected file. |
| Key metadata is missing or incorrect | Create `_metadata.json` before publishing data and declare `keyColumns` for updates, deletes, and upserts. Missing metadata alone does not necessarily prevent insert-only ingestion. |
| `__rowMarker__` is misplaced or invalid | Make it the last column and encode supported numeric values `0`, `1`, `2`, or `4`; the examples use Arrow `int32`. |
| Parquet logical and physical types are incompatible | Cast source values to supported Parquet types before writing. |
| Delimited-text schema does not match `_metadata.json` | Correct `SchemaDefinition`, file extension, delimiters, encoding, and null handling. |
| Sequential file name is invalid or a number is missing | Reconcile the durable batch assignment and restore the exact required 20-digit sequence. Do not blindly rename it to a new number. |
| A column type changed, or a nonnullable column was omitted | Stop publishing incompatible batches and follow the table rebuild procedure if the change is breaking. |

Do not apply the 20-digit filename rule to a table explicitly configured with `LastUpdateTimeFileDetection`. That mode still requires unique immutable paths, and it does not guarantee source event order.

### Issue: Delta table appears empty after upload

**Possible causes**: Replication is not started, capacity is paused, the initial file is missing or invalid, the sequence has a gap, or the SQL endpoint has not synchronised its metadata.

**Resolution**: Check database and table status first. Verify the configured folder, metadata, and first sequential file, `00000000000000000001.parquet`. If the data is already visible through a OneLake shortcut but absent from SQL, use the SQL endpoint **Refresh** action and investigate [metadata synchronisation](https://learn.microsoft.com/en-us/fabric/mirroring/troubleshooting#data-doesnt-appear-to-be-replicating), rather than republishing the source.

### Repairing a Failed File or Rebuilding One Table

The [Open Mirroring recovery guidance](https://learn.microsoft.com/en-us/fabric/mirroring/open-mirroring-best-practices#plan-for-recovery) distinguishes two actions:

- **Failed invalid file**: Delete the zero-byte, corrupt, or otherwise invalid file and atomically publish a complete valid replacement under exactly the same name. In sequential mode, later files wait for that index. This is not permission to overwrite or replay an already processed file. A valid zero-row Parquet file is not a zero-byte file.
- **Table rebuild**: Suspend the table's publisher, delete its entire landing-zone folder, and wait until the mirrored table disappears. Recreate the folder, metadata, and a complete initial load. Resume changes only after the table reappears. Reconcile source capture and sequence state for the new table.

Deleting a table folder drops the mirrored table. Do not use this procedure without a reload plan. Do not stop and restart the entire mirrored database to repair just one table.

---

## Source Watermark and File Ordering Issues

### Issue: Duplicate rows in the Delta table

**Symptoms**: Row counts in Fabric exceed expected counts; duplicate primary keys visible.

**Causes and resolutions:**

| Cause | Resolution |
|---|---|
| Same source changes published in more than one file | Persist the batch-to-sequence assignment before upload. On recovery, verify the existing immutable file, then advance the watermark without republishing the batch. |
| Source extraction position not persisted | Store the watermark or change token durably outside the landing-zone path. |
| Multiple publishers allocate the same or overlapping sequence | Use one coordinated publisher per table and allocate each 20-digit sequence once. |
| Insert marker `0` used for replayable records | Inserts do not check for duplicate keys. Use a unique key and appropriate upserts, while still preventing out-of-order replay. |

### Issue: Missing rows / rows not appearing in Fabric

**Symptoms**: Expected rows are absent from the Delta table.

**Causes and resolutions:**

| Cause | Resolution |
|---|---|
| Watermark advancing past unpublished or late-committing rows | Advance only after verified publication. Timestamp polling also needs tested late-commit handling or reconciliation; a fixed safety lag is not a correctness guarantee. |
| Source query not returning all rows | Verify source query logic; check for implicit filters (e.g., soft-deleted rows excluded). |
| File not fully uploaded before Fabric processing | Write with an underscore prefix, flush it, then atomically rename it to the final sequence. |
| Wrong row operation | Check `keyColumns` and the final `__rowMarker__` value. |
| File sequence has a gap | Restore the missing assigned batch at its original index; later sequential files cannot bypass it. |
| Nonsequential detection applies a late batch over newer state | Review storage `LastModified` order and the producer's ordering design; use sequential detection for order-sensitive changes. |

---

## Diagnosing Backoff and Retry Behaviour

### Issue: Mirroring repeatedly shows Running with warning due to backoff and retries

**Symptoms**: The item's monitoring view shows persistent **Running with warning** status with growing replication lag; table details or logs show repeated retries.

**Diagnostic steps:**

1. Check the error for the specific table in **Monitor replication** and, when enabled, the `MirroredDatabaseTableExecution` workspace monitoring logs.
2. Check source system availability and the publisher's network, authentication, and upload logs. Open Mirroring has no Fabric-managed source connection to repair.
3. Confirm that Fabric capacity is running and check the relevant Fabric/Azure service health information.
4. For API failures, record the HTTP status and request ID. Honour `Retry-After` for throttling instead of immediately resending the request.

**Common causes and resolutions:**

| Cause | Resolution |
|---|---|
| Source database temporarily unavailable | Restore source access and follow that connector's recovery guidance. A custom publisher must implement its own bounded retry policy. |
| Network path blocked | Check the documented gateway, private endpoint, or firewall path for that source. Do not indiscriminately open the source to Fabric IP ranges. |
| Source or publisher credentials expired | Update the native connector's connection or the custom publisher's credential, as appropriate. |
| OneLake upload returns 401/403 | Verify the Storage token audience, identity, and mirrored-item write permission. A Fabric REST token is not a OneLake token. |
| CDC position lost or a replication slot was dropped | Follow source-specific recovery instructions and arrange a consistent reload if required. Recreating a slot alone does not recover missing changes. |
| Rate limiting in a custom extraction API | Honour the source API's retry rules and keep the pending batch assignment durable. |

### Before Stopping and Starting

Do not use stop and start as a routine retry. For Open Mirroring, the published best practices say stopping and restarting the mirrored database restarts it from the beginning. Prepare a complete replay/reload plan; Fabric's processed-file retention is not a source backup. Native database connectors have source-specific reseed behaviour.

Resuming replication after a capacity pause is a separate action from deliberately resetting replication. Check the current state and [capacity troubleshooting guidance](https://learn.microsoft.com/en-us/fabric/mirroring/troubleshooting#changes-to-fabric-capacity) before choosing an action.

---

## Data Consistency

### Issue: Row counts between source and Fabric don't match

**Diagnostic approach:**

1. Compare source and destination at the same stable checkpoint, allowing for replication lag. The portal's **Rows replicated** counts inserts, updates, and deletes processed, not the current row count.
2. Check whether the snapshot has completed. Row counts during the initial snapshot phase are expected to be partial.
3. Verify that all tables show **Running** (not **Running with warning** or **Failed**) status.
4. For Open Mirroring, verify the expected files and durable assignments. For REST monitoring, use table-level states such as `Snapshotting` and `Replicating`, not the portal's labels as API enum values.

**Common causes:**

- Initial snapshot not yet complete.
- Tables showing Running with warning due to retries, with growing lag.
- Unsupported types, schema errors, or connector-specific column exclusions. Do not assume malformed Open Mirroring rows are silently skipped.
- DELETE operations not captured by the source, or Open Mirroring rows do not use `__rowMarker__ = 2`.

### Issue: Data appears stale (old values persisting)

**Cause**: UPDATE operations not being captured correctly.

**Resolution:**
- For Open Mirroring, ensure updated rows contain the complete row and `__rowMarker__ = 1` or `4`.
- For database mirroring, follow the source-specific change-capture configuration checks.
- Verify that `_metadata.json` defines the correct `keyColumns`.
- Check whether an older batch was republished after a newer one. Upsert semantics alone do not prevent stale values from overwriting current values.

---

## Performance Tuning

### Reducing Replication Lag

1. **Measure each stage**: Separate source extraction, upload, and Fabric processing latency.
2. **Avoid tiny files**: Group related changes into practical batches without relying on an undocumented fixed size.
3. **Use atomic delivery**: Partial or repeatedly retried files create avoidable failures.
4. **Parallelise by table**: Extract and upload independent tables concurrently while keeping one ordered publisher per table.

### Reducing Source System Impact

1. **Off-peak extraction**: Schedule extraction windows during source system off-peak hours.
2. **Incremental filtering**: Use efficient watermark queries that leverage indexed columns (avoid full table scans).
3. **Partition pruning**: Where the source supports partitioned tables, ensure watermark queries use the partition column.

### Optimising Query Performance

1. Query only the required columns and filter early.
2. Create SQL views or semantic models for stable consumer logic.
3. If a workload requires physical optimisation or write operations, copy the data into a separate Lakehouse or Warehouse table rather than modifying the mirrored table.

---

## Frequently Asked Questions

**Q: Can I mirror the same source table into multiple Fabric workspaces?**
A: Support depends on the source. Some database connectors allow only one active mirror for a source database. Check the source-specific limitations before creating another mirror.

**Q: Can I write to the same open mirroring landing zone from multiple applications?**
A: Use one coordinated writer per table. Multiple writers can allocate conflicting file sequences or publish source changes out of order.

**Q: What happens to the Delta table if I stop mirroring and then restart it?**
A: Behaviour depends on the mirroring type. Open Mirroring best practices describe a database-wide restart from the beginning; recover one table with the targeted folder rebuild procedure instead. For native connectors, check the source's reseed and recovery documentation.

**Q: How long does the initial snapshot take?**
A: Snapshot duration depends on data volume, source throughput, network transfer, and service processing. Monitor progress in the Fabric portal.

**Q: Can I use the SQL analytics endpoint to modify mirrored table data?**
A: No. The SQL analytics endpoint is read-only for mirrored tables. All data modifications must occur through the source system (for database mirroring) or through the landing zone (for open mirroring).

**Q: Does Fabric Mirroring support schema changes (DDL)?**
A: Support varies by source and file format. Open Mirroring can add Parquet columns and retain omitted nullable columns with nulls for new rows. Type changes and key changes require a table rebuild; delimited-text schema evolution is unsupported. See Chapter 32 and the source-specific chapters.

---

## Summary

Start with monitoring evidence, then investigate source health, connectivity, schema, source checkpoints, and file ordering. For Open Mirroring, validate metadata, the configured detection strategy, immutable atomic publication, and row operations before attempting recovery.

**References:** [General troubleshooting](https://learn.microsoft.com/en-us/fabric/mirroring/troubleshooting), [monitoring](https://learn.microsoft.com/en-us/fabric/mirroring/monitor), [landing-zone requirements](https://learn.microsoft.com/en-us/fabric/mirroring/open-mirroring-landing-zone-format), and [Open Mirroring recovery and schema guidance](https://learn.microsoft.com/en-us/fabric/mirroring/open-mirroring-best-practices).

The project chapters that follow apply this shared guidance to actual source code. Use the [cross-project acceptance exercises](chapter-44.md#acceptance-exercises) to distinguish a successful demonstration from a recoverable replication service.

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 32: Metadata and Change Files](chapter-32.md) | **Next:** [Chapter 34: The Microsoft Open Mirroring Python SDK](chapter-34.md)
