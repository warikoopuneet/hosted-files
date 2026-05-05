← [Back to Executive Summary](executive-summary.md)

## File: gc-pauses.md  

### Problem (Payments Context)  
Java GC pauses can momentarily halt service threads, causing high-latency spikes. In payments, a 100ms pause is unacceptable for real-time auth. Long-lived objects and fragmentation exacerbate this.  

### Progressive Solution Path  
1. **Naïve GC (Default):** Use JVM’s default GC (often ParallelGC or G1). Under heavy load (allocate many objects), STW (stop-the-world) pauses occur.  
2. **Tuning G1:** Set `-XX:MaxGCPauseMillis=10` to target low pauses. Observe trade-off with throughput.  
3. **Concurrent GC:** Switch to Shenandoah or ZGC (Java 15+) which do most work concurrently.  
4. **Object Reuse / Off-Heap:** Use object pools or `ByteBuffer` to reduce heap pressure. Caffeine cache off-heap storage.  
5. **Production Pattern:** High-QPS systems often use ZGC or GraalVM with real-time GC. For example, Google's Ad system uses Chubby tuned for large heaps, but may accept some GC latency.  

### Lab Steps (GC Pauses)  

**Prerequisites:** Docker, Java (for ZGC/Shenandoah require Java 17+).  

1. **Allocate Stress:** A Spring Boot service that processes payments and retains them in a `List` (mimicking cache). Run many inserts to force GC.  
   - **Monitor:** use `jvm_gc_pause_seconds` metric.  
   - **Expect:** Periodic large pauses seen in metrics; during these, no requests are processed (p99 latency spike).  

2. **Enable G1GC Tuning:** Start JVM with `-XX:+UseG1GC -XX:MaxGCPauseMillis=20`.  
   - **Expect:** Pauses become more predictable, likely around 20ms.  
   - **Observe:** Histogram of GC pause should show lower peaks than default.  

3. **Use ZGC/Shenandoah:** Start with `-XX:+UseZGC`.  
   - **Expect:** Almost no visible pauses (<10ms), though CPU usage will be higher.  
   - **Observe:** p99 latency near the baseline (no GC cliffs).  

4. **Object Reuse:** Modify code to reuse DTOs or use primitive arrays for batched data, minimizing allocations.  
   - **Expect:** Lower GC frequency.  

**Observability:**  
- Graph GC pause time (histogram and count).  
- Compare throughput before/after (ZGC may slightly reduce throughput due to more CPU).  

**Verification:**  
- Confirm p99 latency drops when using concurrent GC.  
- Ensure system stability (heap still collected).  

**Checklist:**  
- [ ] Generate GC by load; confirm high p99.  
- [ ] Tune G1GC; observe reduced pauses.  
- [ ] Switch to ZGC; verify minimal GC impact.  
- [ ] Document GC flags and impacts.  

**Sources:** JVM documentation for G1/ZGC. Cassandra notes that major GC or compaction triggers latency spikes【48†L5-L13】.  

---
