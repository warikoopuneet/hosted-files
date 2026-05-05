← [Back to Executive Summary](executive-summary.md)

## File: leader-election-churn.md  

### Problem (Payments Context)  
In replicated state machines (e.g. Raft clusters) backing payment services, frequent leader changes cause temporary unavailability. If a leader’s lease expires due to load, election occurs. During election, no writes are committed, stalling requests. Metrics: leader changes/sec, request failures during elections.  

### Progressive Solution Path  
1. **Naïve (Default Raft):** Launch a 3-node cluster (e.g. etcd). Use default timeouts. Under normal conditions, leader handles writes.  
2. **Induce Churn:** Simulate slow GC (`jcmd GC.run`) on the leader container. The leader misses heartbeats, causing election.  
3. **Timeout Tuning:** Increase election timeout (`--election-timeout` flag) to reduce false elections.  
4. **Sticky Sessions:** Have clients pin to the current leader (after detection) to avoid cross-leader hops.  
5. **Multi-Raft Groups:** Shard the data so each shard has its own independent Raft. Demonstrated conceptually.  
6. **Leaderless Mode:** For read-heavy payments, use a leaderless DB (Cassandra) or read replicas for availability.  

### Lab Steps (Leader Election Churn)  

**Prerequisites:** Docker, etcd or HashiCorp Consul.  

1. **Setup Cluster:** Run 3-node etcd cluster. Write a key via `etcdctl put`.  
2. **Simulate Leadership:** Identify leader (etcdctl endpoint status). Then, trigger a pause (`docker pause`) on leader container for 1s.  
   - **Expect:** A new election occurs (watch logs). Requests during downtime fail or get delayed.  

3. **Tune Election Timeout:** Restart etcd with `--election-timeout=5000ms`. Repeat above test.  
   - **Expect:** Leader holds longer; shorter pause tolerated without election.  

4. **Observe Impact:** Plot request success rate over time. With high churn, see dips; after tuning, stability.  

**Observability:**  
- Export etcd metrics: `etcd_disk_wal_fsync_duration_seconds`.  
- Custom: count failed requests during elections.  

**Verification:**  
- Leader changes count should drop after timeout tuning.  
- No downtime observed for short pauses.  

**Checklist:**  
- [ ] Trigger leader delay; confirm rapid failover.  
- [ ] Increase timeouts; verify leader remains.  
- [ ] Document final cluster config for production use.  

**Sources:** Raft papers advise tuning timers for stable clusters. Metastability studies note leader churn degrades performance【50†L113-L121】.  

---
