
# DARPA FiveDirections E5 → Semantic Endpoint Telemetry Pipeline

## Overview

This project converts raw DARPA Transparent Computing FiveDirections E5 provenance data into semantic endpoint telemetry suitable for behavioral sequence modeling with anomaly detection.

The original dataset contains low-level provenance CDM (Common Data Model) records stored as Avro `.bin.*` files. These records contain fragmented UUID-based graph relationships across multiple object tables.

This pipeline transforms the raw provenance graph into enriched semantic telemetry events similar to OpTC-style endpoint logs.

---

# Dataset Source

Dataset:
- DARPA Transparent Computing
- FiveDirections E5
- CDM provenance records

Raw format:

```text
.bin.*.gz
→ .bin.*
→ Avro CDM records
