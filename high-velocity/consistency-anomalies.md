# Consistency anomalies and isolation levels

## Problem
Weak isolation can produce stale reads, lost updates, non-repeatable reads, or write skew. In payments, that means broken invariants.

## Naive implementation
Read from lagging replicas or update shared state without strict coordination.

## Failure scenario
Run concurrent transfers that should preserve an invariant but do not.

## Solution journey
### 1. Weak reads
Fast, but can be stale.

### 2. Read-your-writes
Route recent reads to the leader or session-local consistent path.

### 3. Snapshot isolation
Avoid dirty reads and reduce blocking.

### 4. Serializable isolation
Prevent the anomaly at the cost of more contention.

### 5. Production-grade design
Use the strictest isolation for money movement, but allow weaker models for reporting or analytics.

## Tradeoffs
- Weak reads are fast but can lie.
- Snapshot isolation improves concurrency but does not stop every anomaly.
- Serializable is safest and most expensive.
- App-level checks help, but should not replace isolation for core ledger correctness.

## Spring Boot lab
### Module
`balance-consistency-service`

### Stages
1. `/read-stale`
2. `/read-your-write`
3. `/txn-snapshot`
4. `/txn-serializable`
5. `/txn-prod`

### Validation
Assert:
- invariant preservation
- no stale balance after committed write
- no lost updates under concurrent transfers

### Final production-grade version
Use strict isolation on the core ledger and read replicas only for non-critical views.
