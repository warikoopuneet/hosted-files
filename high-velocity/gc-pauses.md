# GC pauses and JVM latency

## Problem
The JVM pauses application threads to reclaim memory. In a low-latency payment path, even short pauses can be visible to users.

## Naive implementation
Use default allocation-heavy code and let GC deal with it.

## Failure scenario
Create enough short-lived objects to force visible pauses during load tests.

## Solution journey
### 1. Default GC
Works, but can create tail spikes.

### 2. Tune GC
Choose a collector and tune pause goals.

### 3. Reduce allocations
Reuse objects where safe and reduce temporary garbage.

### 4. Concurrent collectors
Use low-pause collectors for stricter latency targets.

### 5. Production-grade design
Keep the synchronous payment path small and memory-efficient, and push heavy lifting to background workers.

## Tradeoffs
- Lower pauses often mean higher CPU cost.
- Tuning can improve tail latency but requires measurement.
- Object reuse reduces GC pressure but can hurt code clarity.
- Very aggressive latency tuning may reduce throughput.

## Spring Boot lab
### Module
`gc-latency-service`

### Stages
1. `/gc-naive`
2. `/gc-tuned`
3. `/gc-reduced-allocation`
4. `/gc-low-pause`
5. `/gc-prod`

### Validation
Measure:
- GC pause time
- allocation rate
- p99 latency
- CPU overhead

### Final production-grade version
Make the payment API lean, predictable, and allocation-light.
