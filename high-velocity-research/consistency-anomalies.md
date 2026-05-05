← [Back to Executive Summary](executive-summary.md)

## File: consistency-anomalies.md  

### Problem (Payments Context)  
Weak consistency can lead to anomalies: **stale reads** (client sees old balance), **lost updates** (concurrent updates overwrite each other), or **write-skew** (two transactions violate invariants). In payments, this could manifest as double-authorizations or negative balances if isolation is insufficient. Metrics: integrity check failures, anomaly counters.  

### Progressive Solution Path  
1. **Naïve (Eventual Reads):** Expose data from a read-replica (which lags behind the primary). Reads may be stale.  
2. **Read-After-Write Session:** Pin client to primary for a short time after update (read-your-writes).  
3. **Snapshot Isolation:** Use database snapshots; allows some concurrency but avoids dirty reads.  
4. **Serializable Isolation:** Force serial order (e.g. SERIALIZABLE setting in PostgreSQL). Ensures no anomalies.  
5. **App-level Checks:** Implement reconciliation logic to catch anomalies (e.g. verify sum of debits equals expected).  
6. **CRDTs for Non-critical Data:** For append-only logs or metrics, use conflict-free CRDT types to avoid anomalies entirely (not money!).  

**Production Pattern:** Systems that require correctness (e.g. bank ledgers) use serializable transactions or strict locking. Eventually-consistent systems (like Amazon’s shopping cart) accept rare anomalies【44†L61-L69】.  

### Lab Steps (Consistency Anomalies)  

**Prerequisites:** Docker, Java.  

1. **Stale Read Demo:** Use a multi-threaded test: Thread1 writes a payment (updates balance in master), Thread2 immediately reads from a read-replica (or from a cache with slight delay). Show that Thread2 sees old balance (stale).  
   - **Fix:** Instead read from the leader or use `READ COMMITTED` with replica lag awareness.  

2. **Lost Update (Write Skew):** Two concurrent transfers: T1 moves $60 from A→B, T2 moves $60 from B→A. Use SNAPSHOT isolation.  
   - **Observation:** Under SI, both see $100 and both commit, resulting in $40/$160 instead of $100/$100 (invariant broken).  
   - **Fix:** Run under `SERIALIZABLE` isolation or use `SELECT FOR UPDATE` on both rows. Retry one transaction on failure.  

3. **Phantom Read (Cumulative):** Assume an account with multiple sub-balances. T1 transfers between sub-balances; T2 concurrently sums them. Show inconsistent sum under weak isolation. Fix with higher isolation or explicit locks.  

4. **Production Case (Outcomes):**  
   - Example: Financial ledgers (exchange settlement) must use serializable or consensus to prevent double-spend.  
   - E-commerce cart (Spotify example by AWS) allow occasional inconsistencies if user refreshes rapidly【44†L61-L69】 (oversold items due to eventual consistency).  

**Observability:** Check for anomalies: e.g., assert `balanceA+balanceB == constant`. Count violations.  

**Verification:**  
- Show anomalies under lower isolation.  
- After enforcing serializable or locking, confirm invariant holds.  

**Checklist:**  
- [ ] Create scenario with concurrent updates; detect anomaly.  
- [ ] Apply stricter isolation; verify correctness.  
- [ ] Ensure system performance under strict mode is acceptable for the workload.  

**Sources:** ANSI SQL defines anomalies (dirty, non-repeatable reads). Eventual consistency can oversell inventory【44†L61-L69】. Strong systems (Cockroach, Spanner) use serializable guarantees to avoid these.  

---
