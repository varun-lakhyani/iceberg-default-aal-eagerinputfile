# Iceberg EagerInputFile — Benchmark Results

Raw S3 access logs and JMH benchmark output for the `EagerInputFile` optimization on Apache Iceberg compaction, comparing it against the default and the S3 Analytics Accelerator Library (AAL).

---

### Benchmark Setup (JMH)

**Machine:** AWS EC2 (same region as the S3 bucket to minimize network latency)

| Property | Value |
|----------|-------|
| Instance type | r5.4xlarge |
| vCPUs | 16 |
| Memory | 128 GB |
| Network | Up to 10 Gbps |
| Storage | 50 GB gp3 EBS |
| AMI | Amazon Linux 2023 |
| Region | ap-south-1 (Mumbai) |

**Benchmark:** [IcebergDataCompactionBenchmark](https://github.com/apache/iceberg/blob/839b22647ce5aa08e27220bbe85cde1292129fea/spark/v4.1/spark/src/jmh/java/org/apache/iceberg/spark/action/IcebergDataCompactionBenchmark.java#L57) extended with `aalEnabled` and `eagerThreshold` params for this comparison
- Warmup iterations: **5**
- Measurement iterations: **10**
- Total rows: **20,000,000 (20 million)**

---

### Results

| Number of Files | Default (s) | AAL = true (s) | EagerInputFile = true (s) | EagerInputFile Improvement vs Default | EagerInputFile Improvement vs AAL |
|----------------:|------------:|---------------:|--------------------------:|--------------------------------------:|----------------------------------:|
| 250  | 45.104 | 29.671 | 27.311 | **39.45%** | **7.95%** |
| 500  | 77.788 | 54.030 | 46.553 | **40.15%** | **13.84%** |
| 1000 | 163.238 | 107.138 | 95.429 | **41.54%** | **10.93%** |
| 2000 | 312.252 | 195.158 | 179.351 | **42.56%** | **8.10%** |

#### Graphical Comparison

<img width="100%" alt="iceberg_compaction_runtime_axis" src="https://github.com/user-attachments/assets/469dedde-4693-4620-861b-b92e770764d5" />

---

### S3 Request Analysis

The number of GET/HEAD requests observed from the S3 bucket logs is computed as:
> `Total Requests / (Total Iterations × Number of Files)`

| Configuration | GET/file | HEAD/file | Total Calls/file |
|---------------|:--------:|:---------:|:----------------:|
| Default | 3 | 0 | 3 |
| AAL = true | 1 | 1 | 2 |
| EagerInputFile = true | 1 | 0 | **1** |

Full per-file breakdown across all file counts: [network-calls-per-file.md](network-calls-per-file.md)

---

### Raw logs and Results 

| File | Description |
|------|-------------|
| [jmh-benchmark-results.txt](jmh-benchmark-results.txt) | Raw JMH text output |
| [network-calls-per-file.md](network-calls-per-file.md) | S3 GET/HEAD call counts derived from bucket logs |
| [raw-s3-access-logs/](raw-s3-access-logs/) | Raw S3 server access logs (sensitive fields redacted) |