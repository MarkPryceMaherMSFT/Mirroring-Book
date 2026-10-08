# Chapter 45: File Publishing and Open Mirroring Test Tools

> **Part 3: Open Mirroring**
>
> **Purpose:** Use public file publishers, synthetic workloads and sink tests without misclassifying them as production database connectors.

**Part index:** [Chapters in Part 3](readme.md)

---

## An Interactive Excel Publisher

[Fabric-OpenMirroring-for-files](https://github.com/nikunj11itdhm/Fabric-OpenMirroring-for-files) provides a Streamlit interface for workbook ingestion. The inspected revision is `1ce00cc`, with an [MIT licence](https://github.com/nikunj11itdhm/Fabric-OpenMirroring-for-files/blob/1ce00cc401fde89404d981b620179184bbb9de55/LICENSE).

```text
Workbook upload -> archive original in a Lakehouse
    -> worksheet DataFrames -> Parquet -> mirrored database
```

The workbook basename becomes a schema and each worksheet becomes a table. Follow the [README](https://github.com/nikunj11itdhm/Fabric-OpenMirroring-for-files/blob/1ce00cc401fde89404d981b620179184bbb9de55/README.md) for Python dependencies and resource configuration, then run the Streamlit application on an approved host. Configure both the archive destination and mirrored database deliberately; they are different resources.

The [application](https://github.com/nikunj11itdhm/Fabric-OpenMirroring-for-files/blob/1ce00cc401fde89404d981b620179184bbb9de55/app.py) generates positional `__rowid__` values. Clean mode drops/recreates the destination; nonclean mode publishes current rows but does not infer missing-row deletes. Reordering or shortening a workbook therefore needs explicit business-key and reconciliation design.

Its [publishing helper](https://github.com/nikunj11itdhm/Fabric-OpenMirroring-for-files/blob/1ce00cc401fde89404d981b620179184bbb9de55/openmirroring_operations.py) uses a temporary name and checks REST rename status. That is useful, but still does not supply a durable source-batch assignment or snapshot-difference engine. The Microsoft attribution in the helper also means this should not be counted as a newly independent Microsoft SDK.

## Synthetic CRUD with OpenMirroringFaker

[OpenMirroringFaker](https://github.com/lmoloney/OpenMirroringFaker) is an [MIT-licensed](https://github.com/lmoloney/OpenMirroringFaker/blob/main/LICENSE) Python tool with an `omf` CLI and YAML-driven scenarios. It generates inserts, updates and deletes, uses configured keys and writes typed Parquet. Its usefulness is reproducible protocol exercises, not connection to an operational database.

The [generator](https://github.com/lmoloney/OpenMirroringFaker/blob/main/src/open_mirroring_faker/data_generator.py) retains generated row populations and counters in memory. Restarting it is not durable continuation of a source history. Its OneLake writer uses GUID filenames with timestamp-based detection; test ordering rather than assuming GUID uniqueness provides source order.

Use an isolated destination and a repeatable seed to exercise CRUD, nulls and types. Record the expected final rows separately from the publisher's own logs.

## Shape: A Reusable Sink and Contract Tests

[sqllocks/shape](https://github.com/sqllocks/shape) includes an [MIT-licensed](https://github.com/sqllocks/shape/blob/6c88537c0961a0068bbcd289a800dbcc13f7ec6b/LICENSE) `fabric-mirror` sink. The inspected early-access version is 0.9.1 at revision `6c88537`.

The [sink](https://github.com/sqllocks/shape/blob/6c88537c0961a0068bbcd289a800dbcc13f7ec6b/src/shape/builtins/sinks/fabric_mirror.py) accepts Arrow batches, supports Parquet and CSV, validates markers, places the marker last and requires keys for noninsert operations. Read its [usage guide](https://github.com/sqllocks/shape/blob/6c88537c0961a0068bbcd289a800dbcc13f7ec6b/docs/FABRIC_MIRROR.md) for options and Azure dependencies.

Its [contract tests](https://github.com/sqllocks/shape/blob/6c88537c0961a0068bbcd289a800dbcc13f7ec6b/tests/io/test_fabric_mirror.py) are useful examples for metadata, marker placement, file numbering and failure cleanup. Mocked filesystem tests, however, cannot establish that remote `fs.mv` becomes an atomic ADLS Gen2 rename. Verify the actual storage adapter rather than relying on a method name.

A sink handles supplied batches. It does not establish the caller's source snapshot boundary, checkpoint, deletion capture or source-history recovery.

## Benchmarking Without Confusing Measurements

[fabric-open-mirroring-benchmark](https://github.com/mdrakiburrahman/fabric-open-mirroring-benchmark) is an [MIT-licensed](https://github.com/mdrakiburrahman/fabric-open-mirroring-benchmark/blob/main/LICENSE) workload harness. Its Python path uses DuckDB-generated data and publication/visibility measurements; the [instructions](https://github.com/mdrakiburrahman/fabric-open-mirroring-benchmark/blob/main/projects/python/README.md) identify dependencies and configuration. Its publishing helper derives from Toolbox, so count the benchmark harness, not another independent SDK.

Keep four measurements separate: generated rows, uploaded files/bytes, applied Delta changes and SQL-visible results. No throughput number is transferable without the workload, configuration, concurrency and observation method.

Run benchmarks only on capacity and source resources approved for load testing. Do not present synthetic throughput as proof of snapshot consistency or exactly-once recovery.

## A Reusable Lab

Start with stable business keys, known initial rows and a declared schema. Apply an update, insert and delete, then interrupt publication and rerun the same assigned batch. Exercise an empty input, a key change, a new column and several changes to one key.

Compare destination values after Fabric ingestion, not just immediately after upload. Use [Chapter 46's acceptance exercises](chapter-46.md#acceptance-exercises) to expand this into a project qualification plan. Tools can help generate the evidence, but cannot replace a correctly designed source and publication boundary.

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 44: Synapse Dedicated SQL Pool Open Mirroring](chapter-44.md) | **Next:** [Chapter 46: Choosing a Solution, Shared Lessons, and Further Reading](chapter-46.md)
