# Chapter 34: The Microsoft Open Mirroring Python SDK

> **Part 3: Open Mirroring**
>
> **Purpose:** Use Microsoft's Python helper to understand table creation and file publication, while keeping source capture and reliable recovery in your application.

**Part index:** [Chapters in Part 3](readme.md)

---

## What the SDK Provides

The [Open Mirroring Python SDK](https://github.com/microsoft/fabric-toolbox/tree/main/tools/OpenMirroringPythonSDK) was developed inside Microsoft and published in Fabric Toolbox. Its implementation is the `OpenMirroringClient` class in one Python module, `openmirroring_operations.py`. It is not a complete database connector or a replacement for the [shared build guidance](chapter-29.md).

| Responsibility | SDK or application? |
|---|---|
| Authenticate with a client secret and address OneLake | SDK |
| Create table folders and key metadata | SDK |
| Discover a next filename and upload through a temporary name | SDK |
| Read monitoring files | SDK; the inspected methods print their results |
| Extract source rows, capture deletes and establish a snapshot boundary | Application |
| Generate and validate Parquet, maintain durable checkpoints, coordinate writers | Application |
| Create/start the Fabric mirrored database and operate a scheduler | Separate Fabric APIs and application |

**Source baseline:** [implementation at `b4636ef`](https://github.com/microsoft/fabric-toolbox/blob/b4636ef9cb3d6a26863ac5c93480b6c98e9df5a1/tools/OpenMirroringPythonSDK/openmirroring_operations.py), reviewed for this chapter on 8 October 2026. Its [MIT licence](https://github.com/microsoft/fabric-toolbox/blob/b4636ef9cb3d6a26863ac5c93480b6c98e9df5a1/tools/OpenMirroringPythonSDK/LICENSE.txt) permits reuse subject to its terms. Microsoft authorship does not create a production support guarantee; see the [Toolbox support statement](https://github.com/microsoft/fabric-toolbox#support).

## Prepare a Small Lab

Create a new Open Mirrored Database and grant the publishing identity the permissions described in [Chapter 29](chapter-29.md). Use its complete landing-zone URL, not a Lakehouse URL or the SQL endpoint.

Obtain the module from the pinned source and place it on your Python import path. The inspected folder is a source-module distribution; do not invent a `pip install OpenMirroringPythonSDK` package. Install its dependencies and a Parquet library for the example:

```powershell
python -m pip install azure-identity azure-storage-file-datalake requests pyarrow
```

For reproducible deployments, record the tested dependency versions rather than continually installing the latest versions. Store credentials in protected configuration; the example reads environment variables so no credential is embedded in source.

```python
import os
import pyarrow as pa
import pyarrow.parquet as pq
from openmirroring_operations import OpenMirroringClient

client = OpenMirroringClient(
    client_id=os.environ["AZURE_CLIENT_ID"],
    client_secret=os.environ["AZURE_CLIENT_SECRET"],
    client_tenant=os.environ["AZURE_TENANT_ID"],
    host=os.environ["FABRIC_LANDING_ZONE_URL"],
)

client.create_table(
    schema_name="demo", table_name="customers", key_cols=["id"]
)

schema = pa.schema([
    pa.field("id", pa.int64(), nullable=False),
    pa.field("name", pa.string()),
])
snapshot = pa.Table.from_pylist([
    {"id": 1, "name": "Ada"},
    {"id": 2, "name": "Grace"},
], schema=schema)
pq.write_table(snapshot, "customers-initial.parquet")
client.upload_data_file(
    schema_name="demo",
    table_name="customers",
    local_file_path="customers-initial.parquet",
)
```

Run table creation and this initial-load cell once against a fresh lab table. Do not rerun the whole cell as your incremental scheduler: allocating another marker-free snapshot file can insert duplicates. The SDK does not remember that the source batch has already been delivered.

After inspecting the actual file and Fabric table status, prepare a full-row upsert and a key-based delete:

```python
change_schema = schema.append(
    pa.field("__rowMarker__", pa.int32(), nullable=False)
)
changes = pa.Table.from_pylist([
    {"id": 1, "name": "Ada Lovelace", "__rowMarker__": 4},
    {"id": 2, "name": None, "__rowMarker__": 2},
], schema=change_schema)
pq.write_table(changes, "customers-changes.parquet")
client.upload_data_file(
    schema_name="demo",
    table_name="customers",
    local_file_path="customers-changes.parquet",
)
client.get_table_status(schema_name="demo", table_name="customers")
```

The marker is the final column. The delete retains the original typed schema, with a nullable non-key field. Expected final state is key `1` with the updated name and no key `2`. This is a method walkthrough, not a crash-safe publisher: it deliberately does not advance a source checkpoint after an upload call.

## Read the Actual API

| Method | Behaviour in the inspected module |
|---|---|
| `create_table(schema_name=None, table_name="", key_cols=[])` | Creates the folder and `_metadata.json` containing `keyColumns`; it does not validate the payload schema |
| `get_next_file_name(schema_name=None, table_name="")` | Finds final `.parquet` names and returns a 20-digit next name |
| `upload_data_file(schema_name=None, table_name="", local_file_path="")` | Reads the local file into memory, uploads under an underscore-prefixed name, flushes, and invokes REST rename |
| `get_mirrored_database_status()` | Reads and prints `Monitoring/replicator.json` |
| `get_table_status(schema_name=None, table_name=None)` | Reads and prints `Monitoring/tables.json`; filtering expects both schema and table |
| `remove_table(schema_name=None, table_name="", remove_schema_folder=False)` | Deletes a table folder; the optional schema-folder deletion can affect other tables |

Use keyword arguments: a positional table name could accidentally be interpreted as `schema_name`. Status methods do not return dictionaries. Do not build code around an invented success result from upload, either.

## Limitations That Matter

The [rename implementation](https://github.com/microsoft/fabric-toolbox/blob/b4636ef9cb3d6a26863ac5c93480b6c98e9df5a1/tools/OpenMirroringPythonSDK/openmirroring_operations.py#L201-L226) prints non-success HTTP responses instead of raising an exception. The upload caller can subsequently print a success message. Treat this as an implementation issue to correct before using method completion as permission to acknowledge source changes.

Filename discovery is not reservation. Two writers can choose the same next filename; a crash between publication and checkpointing can replay a batch under a different filename. Preserve a durable batch-to-path assignment, enforce immutable final paths, propagate errors and reconcile ambiguous outcomes as described in [Chapter 29](chapter-29.md). Ordinary cleanup retains the latest sequential file, but that does not replace your journal.

The module does not generate Parquet, validate markers, configure CSV metadata or expose the alternative file-detection strategy. An arbitrary local file can receive a `.parquet` destination name. Its raw rename request also lacks an explicit timeout. Large-file memory use, retry policy, structured monitoring results and credential flexibility are application-hardening work.

Prefer the documented [Fabric monitoring APIs](https://learn.microsoft.com/en-us/rest/api/fabric/mirroreddatabase/mirroring) for operational automation. A successful upload and a replicated table are different states.

## What to Learn

This is a compact, useful reference for the OneLake side of a publisher. Reuse its understandable structure, not assumptions about end-to-end reliability. Before deployment, exercise failed rename, response loss, restart after publication, delete processing and two competing writers. Deleting a folder is a table-drop operation, not routine cleanup.

**References:** [SDK source and README](https://github.com/microsoft/fabric-toolbox/tree/main/tools/OpenMirroringPythonSDK), [landing-zone contract](https://learn.microsoft.com/en-us/fabric/mirroring/open-mirroring-landing-zone-format), and [publication/recovery guidance](https://learn.microsoft.com/en-us/fabric/mirroring/open-mirroring-best-practices).

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 33: Common Issues and Troubleshooting](chapter-33.md) | **Next:** [Chapter 35: GenericMirroring - A Multi-Source C# Publisher](chapter-35.md)
