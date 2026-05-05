← [Back to Executive Summary](executive-summary.md)

## File: contention.md  

### Problem (Payments Context)  
When multiple clients or services concurrently **debit the same account**, race conditions can lead to **double-charging or lost updates**【60†L118-L127】. For example, two payment attempts on the same invoice might both succeed if unchecked, overshooting the balance. Metrics: high lock wait times, conflicting update errors, or inconsistent balances. A naive implementation (no locks) will allow these anomalies (e.g. final balance incorrect).  

### Progressive Solution Path  
1. **Naïve (No Locking):** A simple Spring Boot `@RestController` debits an account record without any concurrency control. Under load, this will fail (balances become incorrect).  
2. **Add Idempotency Key:** Introduce a unique **idempotency key** in the payment request. Store processed IDs and ignore duplicates【71†L398-L407】. This avoids double processing of retries but does NOT prevent two *distinct* concurrent requests from both succeeding.  
3. **Pessimistic Locking:** Use database locking (e.g. `SELECT ... FOR UPDATE`) on the account row. In JPA: `repo.findById(id, LockModeType.PESSIMISTIC_WRITE)`. This serializes updates【60†L118-L127】. It ensures correctness but under high concurrency will create queuing.  
4. **Optimistic Concurrency (Versioning):** Add a `@Version` field on the account entity. Transactions fail if the version changed, requiring retries【68†L77-L86】. This allows higher throughput with retries. DynamoDB implements this via conditional writes【68†L77-L86】.  
5. **Multi-Version Concurrency (MVCC):** Let the DB provide snapshot isolation (e.g. SERIALIZABLE) so each transaction sees a consistent snapshot. Conflicts cause rollbacks. This is similar to OCC but managed by the DB.  
6. **Sharding Partitioning:** Horizontally partition accounts (e.g. userID % N) so that contention is per-shard. One hot account only locks its shard. This scales throughput by parallelism.  
7. **CRDT (Advanced/Experimental):** For commutative operations (e.g. credit points), use CRDTs. For currency, CRDTs aren’t directly applicable since operations must be serialized.  

**Production Pattern:** Payment APIs must be *idempotent*. Stripe’s API uses idempotency keys for charge endpoints【71†L398-L407】 so retries do not double-charge. Uber’s payment service records each change in an append-only log with version numbers per user【65†L171-L179】, guaranteeing consistent ordering under concurrency.  

```mermaid
flowchart LR
  User1 -->|POST /pay| API[Payment API (Spring Boot)]
  User2 -->|POST /pay| API
  API --> PaymentSvc[Payment Service]
  PaymentSvc --> LedgerSvc[Ledger Service]
  LedgerSvc --> DB[(Accounts Database)]
```
*Figure: Two clients concurrently call a Payment Service that debits an account in the DB. Without control, race conditions occur.*  

### Lab Steps (Contention)  

**Prerequisites:** Java (17+), Docker, Docker Compose, k6 or JMeter.  

1. **Naïve Implementation:** Create a Spring Boot app (`naive` project) with a `PaymentController` that directly subtracts from an account table. No versioning or locks. Deploy via Docker-compose with H2 or Postgres.  
   - **Expect:** Under concurrent load (k6 script sending many `/pay` to same account), you will observe inconsistent final balances (double debits). Prometheus/Grafana shows high rates of `service_response_time_ms` and no retry metrics.  
   - **Verification:** After load (50 concurrent VUs for 10s), check DB: balance is wrong (could be negative).  

2. **Idempotency Key:** Modify the controller to require a client-provided idempotency key. Store processed keys in DB. Ignore repeats.  
   - **Code:** Add `idempotencyKey` header; use a table to track keys. If duplicate, return previous result without reapplying debit【71†L398-L407】.  
   - **Expect:** Retried requests with same key do not double-charge. However, *two distinct requests* still both succeed (this only solves client-side retries).  
   - **Metrics:** Add a counter for idempotent conflicts.  

3. **Pessimistic Locking:** Update the service to use `@Transactional` and lock the account row. In JPA:  
   ```java
   Account acct = repo.findById(accountId, LockModeType.PESSIMISTIC_WRITE);
   acct.setBalance(acct.getBalance().subtract(amount));
   repo.save(acct);
   ```  
   - **Expect:** Under concurrency, requests queue at the DB. Final balance remains correct. Throughput drops. Grafana shows increased request durations.  
   - **Verification:** All `/pay` requests succeed serially, never overshooting balance. Lock contention metrics (from DB) increase.

4. **Optimistic Concurrency:** Remove explicit locks; add `@Version int version;` to `Account` entity (JPA does it automatically on save). Catch `OptimisticLockException` and retry or fail gracefully.  
   - **Code:**  
   ```java
   @Entity class Account {
       @Id Long id; BigDecimal balance;
       @Version int version;
   }
   ```  
   - **Expect:** Under load, some requests will fail with version conflict. Throughput is higher than locking, but some payments report conflict. Total withdrawn ≤ expected.  
   - **Metrics:** Count of conflicts vs successes.  

5. **Production-grade (Sharded):** For high-volume use-cases, introduce sharding. Partition accounts by hash; deploy multiple DB instances. Use a consistent hash router.  
   - **Setup:** In Docker Compose, spin up 2 DBs (ShardA, ShardB) and route based on accountId parity.  
   - **Expect:** Load spreads across shards. Each shard sees much less contention, and overall throughput rises.  

**Observability:** For each stage, collect: request rate (k6), error/retry counts, latency histograms, and business metrics (successful payments, conflict errors). Grafana dashboards visualize p50/p95 latencies and compare “naïve vs locked vs optimistic vs sharded”.  

**Expected Results:**  

| Stage             | Throughput (req/s) | p99 Latency | Conflicts or Errors | Balance Consistency |
|-------------------|--------------------|-------------|---------------------|---------------------|
| Naïve (no-lock)   | Highest            | Lowest      | Wrong (overshoots)  | *Incorrect*         |
| Idempotent        | High               | Low         | None (retries safe) | *Incorrect*         |
| Pessimistic Lock  | Low                | High        | 0 (blocked serial)  | Correct            |
| Optimistic CC     | Medium             | Medium      | Medium (retries)    | Correct after retries |
| Sharded (Prod)    | Highest (parallel) | Low         | Minimal (per-shard) | Correct            |

**Rollback/Cleanup:**  
- Stop Docker Compose, drop databases.  
- Remove any test data (k6 script can cleanup each run).  

**Checklist:**  
- [ ] Naïve run: confirm double-charges occur.  
- [ ] Apply idempotency key: verify retry-safe.  
- [ ] Use pessimistic lock: verify mutual exclusion, lower throughput.  
- [ ] Use optimistic lock: verify final totals correct, measure retries.  
- [ ] Shard accounts: verify balanced load, max throughput.  

**Sources:** AWS DynamoDB uses conditional writes (optimistic locking) under the hood for counters【68†L77-L86】. Stripe explicitly recommends idempotent APIs for payments【71†L398-L407】. Uber’s payment ledger uses versioned entity logs for consistency under concurrency【65†L171-L179】. These industrial patterns align with our lab progression.

---
