← [Back to Executive Summary](executive-summary.md)

## File: resource-exhaustion.md  

### Problem (Payments Context)  
Payment APIs may consume excessive CPU, memory, or file handles under peak load. For example, unbounded thread creation or leaks in caching can exhaust the JVM heap, causing long GC or crashes. Metrics: high CPU (100% per core), memory usage nearing limits, GC time skyrockets, or errors like “too many open files”.  

### Progressive Solution Path  
1. **Naïve (No Limits):** Use default thread-per-request (Tomcat’s max threads maybe large) and no connection pools. Handle thousands of concurrent clients directly.  
2. **Connection/Thread Pools:** Configure sensible limits (e.g. max 200 Tomcat threads, 10 DB connections). Use Spring’s `TaskExecutor` with fixed thread pool.  
3. **Bulkheads:** Segregate resources per subsystem. E.g., dedicate threads to payment processing vs reporting. Use Semaphores/Bulkhead from Resilience4j.  
4. **Heap Tuning:** Adjust `-Xmx` and GC. Monitor with profiler; fix leaks (e.g. caches clearing).  
5. **Production Pattern:** Rate-limit at the edge (API gateway), and decouple heavy tasks (reporting) into offline jobs (like Hive/Spark).  

### Lab Steps (Resource Exhaustion)  

**Prerequisites:** Docker, Java.  

1. **High Concurrency:** Spring Boot *UploadService* that handles file uploads. Default uses Tomcat threads. Use a script to open 1000 concurrent HTTP connections.  
   - **Expect:** CPU 100%, full thread pool, large heap. Prometheus: `process_cpu_seconds` maxed, `jvm_memory_used` rising.  

2. **Thread Pool Limit:** Set `server.tomcat.max-threads=200`. Add a thread dump link.  
   - **Expect:** New connections queue at OS/Tomcat level; some get rejected or timed out (502). CPU uses all 200 threads.  

3. **Bulkhead for Async Tasks:** Suppose service calls an external auditor (simulate slow); use Resilience4j Bulkhead to limit concurrent calls to 5. Other requests wait in queue (or fail fast).  
   - **Expect:** Only 5 concurrent outgoing calls; prevents thread pile-up on downstream calls.  

4. **Leak Fix:** Introduce a deliberate resource leak (e.g. not closing a Stream). Show metric (`process_open_fds`). Then fix by closing in finally.  
   - **Expect:** Open FD count stops growing after fix.  

**Observability:**  
- Use JVM metrics: `jvm_memory_bytes_used`, `tomcat_threads_current`.  
- Grafana: show thread count and memory over time.  

**Verification:**  
- With limits: show stable memory vs unbounded.  
- With bulkheads: verify limited in-flight calls.  

**Checklist:**  
- [ ] Simulate thousands of requests; observe max threads.  
- [ ] Apply thread limits; ensure service still runs stably.  
- [ ] Introduce and fix a file descriptor leak; monitor `fd` count.  

**Sources:** Bulkhead and connection-pool patterns are recommended for resilient services. Netflix’s Design Docs discuss thread isolation to prevent resource overrun.  

---
