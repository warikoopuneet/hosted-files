← [Back to Executive Summary](executive-summary.md)

## File: scalability-limits.md  

### Problem (Payments Context)  
Even with all fixes, fundamental scalability limits can appear. This includes Amdahl’s law (serial sections), network bandwidth, or database constraints. Payments often require at least some global coordination (e.g. end-of-day settlement), which doesn’t parallelize.  

### Progressive Solution Path  
1. **Naïve Scaling:** Add more nodes/microservices expecting linear throughput gain.  
2. **Identify Bottlenecks:** Use profiling. Often, serialization occurs at transaction manager or message broker (single partition).  
3. **Parallelize Work:** Re-architect serial tasks (e.g. run reconciliation in parallel threads or data partitions).  
4. **Async Design:** Offload non-critical tasks entirely to async processing (ETL, batch jobs).  
5. **Production Pattern:** Companies move heavy computation (e.g. fraud ML scoring) to distributed systems (Kafka/Storm, Flink).  

### Lab Steps (Scalability Limits)  

**Prerequisites:** Docker, Java.  

1. **Scaling Demo:** Set up an aggregator service that queries 4 worker microservices for data (simulating a global sum).  
2. **Increase Workers:** Run with 1, 2, 4 workers and measure end-to-end latency and throughput of aggregator.  
   - **Expect:** Throughput plateaus as network/aggregation overhead dominates.  
3. **Optimize:** Show (conceptually) using fan-in (e.g. hierarchical aggregation).  

**Observability:**  
- Plot throughput vs worker count.  
- Identify saturation point.  

**Checklist:**  
- [ ] Demonstrate diminishing returns (throughput flattening).  
- [ ] (Optional) Implement partial parallel reduction; show improvement.  

**Sources:** Amdahl’s law and distributed computing research. Netflix’s scale-out blogs note that beyond a point, adding nodes yields minimal gain.  

---
