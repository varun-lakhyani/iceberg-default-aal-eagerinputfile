# S3 Network Calls per File 

**Source:** S3 server access logs at `s3://iceberg-spark-readers-poc-logs/bench-logs/`   
**JMH:** `@Warmup(5)` + `@Measurement(10)` = 15 invocations per cell  
**Math:** `per_file = total / 15 / numFiles` (every value below is an exact integer)

| numFiles | AAL | eager | data\_GET | data\_HEAD | GET/file | HEAD/file | NETWORK\_CALLS/file |
|----------|-----|-------|-----------|------------|----------|-----------|---------------------|
| 250 | OFF | 0 (naive) | 11250 | 0 | 3 | 0 | 3 |
| 250 | OFF | 1 MB | 3750 | 0 | 1 | 0 | **1** ← Eager |
| 250 | ON | 0 | 3750 | 3750 | 1 | 1 | 2 ← AAL |
| 250 | ON | 1 MB | 3750 | 3750 | 1 | 1 | 2 ← both |
| 500 | OFF | 0 (naive) | 22500 | 0 | 3 | 0 | 3 |
| 500 | OFF | 1 MB | 7500 | 0 | 1 | 0 | **1** |
| 500 | ON | 0 | 7500 | 7500 | 1 | 1 | 2 |
| 500 | ON | 1 MB | 7500 | 7500 | 1 | 1 | 2 |
| 1000 | OFF | 0 (naive) | 45000 | 0 | 3 | 0 | 3 |
| 1000 | OFF | 1 MB | 15000 | 0 | 1 | 0 | **1** |
| 1000 | ON | 0 | 15000 | 15000 | 1 | 1 | 2 |
| 1000 | ON | 1 MB | 15000 | 15000 | 1 | 1 | 2 |
| 2000 | OFF | 0 (naive) | 90000 | 0 | 3 | 0 | 3 |
| 2000 | OFF | 1 MB | 30000 | 0 | 1 | 0 | **1** |
| 2000 | ON | 0 | 30000 | 30000 | 1 | 1 | 2 |
| 2000 | ON | 1 MB | 30000 | 30000 | 1 | 1 | 2 |

## Compressed (independent of numFiles)

| Strategy | GETs/file | HEADs/file | Total calls/file |
|----------|-----------|------------|------------------|
| Naive `S3InputStream` | 3 | 0 | 3 |
| Eager alone | 1 | 0 | **1** ← lowest |
| AAL alone | 1 | 1 | 2 |
| Eager + AAL | 1 | 1 | 2 (AAL's HEAD still fires) |

## Conclusions

- Eager cuts read-path S3 traffic to **1/3 of naive** and **1/2 of AAL**.
- Eager + AAL is not additive: AAL's per-file HEAD persists regardless of Eager wrapping.