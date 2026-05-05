# Backpressure, queues, and load shedding

## Problem
The producer sends more payments than downstream workers can safely process. Queues grow, memory fills, and the system can collapse.

## Naive implementation
An unbounded in-memory queue accepts everything.

## Failure scenario
Flood the API with incoming payment requests while slowing the worker thread.

## Solution journey
### 1. Unbounded queue
Easy, but unsafe.

### 2. Bounded queue
Stop the queue from growing forever.

### 3. Rate limiting
Limit request intake before the system is overloaded.

### 4. Load shedding
Reject excess traffic quickly and explicitly.

### 5. Circuit breaker
Stop calling a slow downstream service when it becomes unhealthy.

### 6. Production-grade design
Use durable queues, demand-driven consumers, and clear admission control at the edge.

## Tradeoffs
- Unbounded queues preserve requests until the crash.
- Bounded queues preserve memory but can block callers.
- Rate limiting protects the platform but may reject revenue.
- Load shedding keeps the system alive at the cost of incomplete work.
- Circuit breakers require clients to tolerate temporary failures.

## Spring Boot lab
### Module
`payment-ingest-service`

### Stages
1. `/ingest-naive`
2. `/ingest-bounded`
3. `/ingest-rate-limited`
4. `/ingest-shed`
5. `/ingest-prod`

### Validation
Track:
- queue depth
- accepted vs rejected requests
- worker lag
- memory usage

### Final production-grade version
Use a bounded queue, explicit backpressure, and a durable broker for work that must survive spikes.
