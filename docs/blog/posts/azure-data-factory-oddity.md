---
date:
  created: 2025-04-12
tags:
    - Azure
    - Data Factory
categories:
  - Cloud
---
# Azure Data Factory Data Flow - Oddity of the week

This week, I stumbled into an interesting puzzle while investigating an issue for a client. <!-- more --> They had recently transitioned from System A to System B, and one of their critical metrics suddenly showed significant discrepancies. Business-wise, these numbers were expected to remain identical, so naturally, it called for some digging.

I started by checking the basics:

Source API: Was the new system feeding incorrect data? No issues there.

Transformed Reporting Tables: Were calculations or transformations misconfigured? Again, everything seemed correct.

Prepared Source Table: This is where I found something odd—unexpected null values in a key column.

This discovery puzzled me. The preceding step involved an upsert operation using Azure Data Factory’s data flows. Normally, if there were null values in critical key columns, Data Factory would raise an error. Yet, here was data that clearly broke this assumption.

My next step was checking the staging table. Strangely, there were no nulls there—the data was complete. It wasn’t long before I realized the real issue: data type mismatch. Specifically, the id column in the target table was configured as Int16, whereas the incoming data from the source was Int64. Azure Data Factory's upsert silently converted incompatible values to null rather than throwing an explicit error.

To confirm my hypothesis, I conducted a small test. Changing the data type in my test environment from Int16 to Int64 immediately resolved the problem during the upsert operation.

Leveraging Delta Lake and Polars for Quick Fixes
Since the client’s data architecture was built around a Lakehouse using Delta Tables, I had the flexibility to perform quick fixes directly on my local machine. Instead of spinning up an entire Spark session (slow and resource-intensive), I turned to lighter, faster libraries—Polars and DeltaLake—to resolve the issue efficiently.

Here’s how I handled the cleanup and schema correction with a straightforward Python script:
```python
import os
import polars as pl
from deltalake import DeltaTable

storage_options = {
  "ACCOUNT_NAME": "client-account-name"
  "ACCESS_KEY": os.getenv["ACCESS_KEY"]
}

# get the delta table and delete faulty records
dt = DeltaTable(
  "path/to/table",
  storage_options=storage_options
)

# .delete() accepts SQL clause
dt.delete("key_column is null")

# read the table
df = pl.read_delta(
  "path/to/table",
  storage_options=storage_options
)

# Cast to new type
df = df.cast({"key_column": pl.Int64})

# Overwrite the table and its schema
df.write_delta(
  "path/to/table",
  storage_options=storage_options,
  mode="overwrite",
  delta_write_options={"schema_mode": "overwrite"}
)
```
