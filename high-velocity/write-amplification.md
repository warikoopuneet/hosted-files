# Storage write amplification and compaction

## Problem
Under heavy write load, a ledger engine can write the same logical data multiple times because of WAL flushes, compactions, or page rewrites.

## Naive implementation
Write each payment as an individual record to an LSM-backed store with default settings.

## Failure scenario
Run sustained ingest and watch disk I/O rise faster than useful writes.

## Solution journey
### 1. Default LSM settings
Easy, but can create more background I/O than expected.

### 2. Tune compaction
Adjust memtable size and compaction strategy.

### 3. Batch writes
Group multiple payment events into a single flush or transaction.

### 4. Use a different storage engine
Move a write-heavy, low-latency ledger to a store with lower write amplification characteristics for that workload.

### 5. Production-grade design
Separate the hot write path from analytical and archival storage.
Keep the real-time ledger lean and move historical reporting out of band.

## Tradeoffs
- Tuning compaction reduces I/O but may raise memory use.
- Batching improves throughput but increases latency per item.
- Different storage engines shift complexity into deployment and operations.
- Offloading analytics protects the ledger but introduces duplication.

## Spring Boot lab
### Module
`ledger-store-service`

### Stages
1. `/ledger-naive`
2. `/ledger-batched`
3. `/ledger-tuned`
4. `/ledger-separated`
5. `/ledger-prod`

### Validation
Measure:
- bytes written
- compaction activity
- fsync count
- p99 write latency

### Final production-grade version
Keep a transactional ledger on a tuned primary store and push immutable events to an analytics pipeline.
