# DARPA FiveDirections E5 → Semantic Endpoint Telemetry Pipeline

## Overview

This project converts raw DARPA Transparent Computing FiveDirections E5 provenance data into semantic endpoint telemetry suitable for behavioral sequence modeling with DeepCASE.

The original dataset contains low-level provenance CDM (Common Data Model) records stored as Avro `.bin.*` files. These records contain fragmented UUID-based graph relationships distributed across multiple object tables.

The pipeline transforms the provenance graph into enriched semantic telemetry events similar to OpTC-style endpoint telemetry.

---

# Dataset Information

## Dataset

- DARPA Transparent Computing
- FiveDirections E5
- CDM Provenance Dataset

## Raw Dataset Format

```text
.bin.*.gz
→ decompressed .bin.*
→ parsed Avro CDM records
```

## Collection Window

Start:
- 2019-05-07

End:
- 2019-05-08

## Environment

Source:
- Windows FiveDirections systems

Telemetry Source:
- `SOURCE_WINDOWS_FIVEDIRECTIONS`

---

# Objective

The goal of this project is to convert low-level provenance graph records into semantic endpoint telemetry for:

- DeepCASE preprocessing
- behavioral sequence learning
- anomaly detection
- cyber threat hunting
- endpoint behavior analytics
- semantic event abstraction

---

# Original Problem

Raw DARPA provenance records are difficult to use directly because:

- events reference UUID-based objects
- semantics are distributed across multiple CDM tables
- provenance graphs are not human-readable
- DeepCASE requires semantic event streams rather than raw graph edges

Example raw provenance event:

```text
EVENT_OPEN
predicateObject = 9b8cb52f-80e9-4a2e-a7b3-6447bd67e8eb
```

This does not directly reveal:
- object type
- file path
- process behavior
- semantic meaning

---

# Processing Pipeline

## Step 1 — Dataset Decompression

Raw compressed DARPA archives were decompressed:

```text
.bin.*.gz
→ .bin.*
```

---

## Step 2 — Avro Parsing

CDM Avro records were parsed using:

- Python
- fastavro

Each CDM record was converted into CSV format.

---

## Step 3 — CDM Record Extraction

The following CDM tables were extracted separately:

```text
events/
subjects/
file_objects/
netflow_objects/
memory_objects/
registry_key_objects/
ipc_objects/
srcsink_objects/
```

Large outputs were automatically split into parts to avoid CSV row limitations.

---

# CDM Record Types

## Main Event Stream

| Record Type | Purpose |
|---|---|
| RECORD_EVENT | Main behavioral provenance events |

---

## Supporting Object Tables

| Record Type | Purpose |
|---|---|
| RECORD_SUBJECT | Process and thread metadata |
| RECORD_FILE_OBJECT | File object information |
| RECORD_NET_FLOW_OBJECT | Network flow metadata |
| RECORD_MEMORY_OBJECT | Memory allocation information |
| RECORD_REGISTRY_KEY_OBJECT | Registry activity |
| RECORD_IPC_OBJECT | IPC communication |
| RECORD_SRC_SINK_OBJECT | Source/sink endpoint semantics |

---

## Ignored Records

| Record Type | Reason |
|---|---|
| RECORD_TIME_MARKER | Timestamps already available in RECORD_EVENT |

---

# Event-Centric Enrichment

`RECORD_EVENT` was selected as the primary telemetry stream.

Each event contains:

- event type
- subject UUID
- predicateObject UUID
- timestamps
- optional paths
- event properties

Example:

```text
EVENT_OPEN
subject = process UUID
predicateObject = object UUID
```

---

# UUID Resolution

The pipeline resolves:

```text
predicateObject
```

against multiple CDM object tables:

- file objects
- network objects
- memory objects
- registry objects
- IPC objects
- source/sink objects

This converts unresolved provenance references into semantic object categories.

---

# Semantic Abstraction

Low-level provenance events were transformed into semantic behavioral templates.

## Examples

| Raw Provenance | Semantic Event |
|---|---|
| EVENT_OPEN + FILE | PROCESS_OPEN_FILE |
| EVENT_WRITE + FILE | PROCESS_WRITE_FILE |
| EVENT_SENDTO + NET_FLOW | PROCESS_SENDTO_NET_FLOW |
| EVENT_CREATE_OBJECT + MEMORY | PROCESS_CREATE_OBJECT_MEMORY |
| EVENT_CHECK_FILE_ATTRIBUTES + FILE | PROCESS_CHECK_FILE_ATTRIBUTES_FILE |

---

# Fallback Semantic Rules

Some provenance objects could not be directly resolved through UUID joins.

Behavioral fallback rules were added to infer object types.

---

## File Object Detection

If:
- `predicateObjectPath` exists
- OR event belongs to file-like operations

Then:

```text
object_type = FILE
```

### File-Like Events

```text
EVENT_OPEN
EVENT_READ
EVENT_WRITE
EVENT_CLOSE
EVENT_CHECK_FILE_ATTRIBUTES
EVENT_MODIFY_FILE_ATTRIBUTES
EVENT_RENAME
EVENT_UNLINK
EVENT_DELETE
EVENT_TRUNCATE
```

---

## Network Object Detection

If event belongs to:

```text
EVENT_SENDTO
EVENT_RECVFROM
EVENT_CONNECT
EVENT_ACCEPT
EVENT_BIND
EVENT_LISTEN
```

Then:

```text
object_type = NET_FLOW
```

---

## Registry Object Detection

If path contains:

```text
REGISTRY
```

Then:

```text
object_type = REGISTRY
```

---

# Final Semantic Telemetry

The final dataset represents endpoint telemetry rather than raw provenance graphs.

---

## Final Dataset Columns

| Column | Description |
|---|---|
| timestampNanos | Event timestamp |
| datetime_utc | Human-readable timestamp |
| hostId | Host identifier |
| subject | Process/thread UUID |
| subject_cid | Process identifier |
| event_type | Original CDM event |
| predicateObject | Referenced object UUID |
| object_type | Resolved semantic object category |
| object_value | Semantic object value |
| semantic_event | Final behavioral event template |
| properties | Original CDM event metadata |

---

# Example Final Events

```text
PROCESS_OPEN_FILE
PROCESS_WRITE_FILE
PROCESS_SENDTO_NET_FLOW
PROCESS_CREATE_OBJECT_MEMORY
PROCESS_CHECK_FILE_ATTRIBUTES_FILE
```

---

# Example Enriched File Event

## Raw CDM Event

```text
EVENT_OPEN
predicateObject = UUID
```

## Resolved Event

```text
PROCESS_OPEN_FILE
```

## Enriched Object Value

```text
\Device\HarddiskVolume2\Windows\Fonts\framd.ttf
```

---

# Example Enriched Network Event

## Raw CDM Event

```text
EVENT_SENDTO
predicateObject = UUID
```

## Resolved Event

```text
PROCESS_SENDTO_NET_FLOW
```

---

# OpTC-Style Endpoint Telemetry

The final enriched dataset resembles OpTC/FiveDirections endpoint telemetry abstraction.

## Mapping

| OpTC Concept | Current Dataset |
|---|---|
| action | semantic_event |
| actorID | subject |
| object | object_type |
| objectID | predicateObject |
| pid | subject_cid |
| timestamp | datetime_utc |
| context | object_value / properties |

---

# DeepCASE Integration

The enriched semantic telemetry dataset is suitable for:

- DeepCASE preprocessing
- behavioral sequence generation
- semi-supervised anomaly detection
- cyber threat behavior modeling

---

## Recommended DeepCASE Input Columns

Minimal format:

```text
timestampNanos
hostId
subject
semantic_event
```

Optional enriched format:

```text
timestampNanos
hostId
subject
semantic_event
object_type
object_value
```

---

# Technologies Used

- Python
- pandas
- fastavro
- UUID resolution
- semantic enrichment pipeline

---

# Key Contribution

This workflow converts:

```text
raw DARPA provenance graphs
```

into:

```text
semantic behavioral endpoint telemetry
```

bridging the gap between:

- provenance-based cyber datasets
- endpoint telemetry analytics
- semantic event abstraction
- sequence-based anomaly detection systems like DeepCASE

---

# Final Outcome

The resulting dataset is:

- human-readable
- semantically enriched
- behavior-oriented
- sequence-ready
- compatible with DeepCASE preprocessing

instead of raw UUID-based provenance graph edges.
