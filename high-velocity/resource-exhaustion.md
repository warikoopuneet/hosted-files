# Resource exhaustion and bulkheads

## Problem
Threads, memory, sockets, file handles, and database connections can all be exhausted by high concurrency.

## Naive implementation
Use default server settings and no bulkhead isolation.

## Failure scenario
Spike the API with many concurrent calls and observe thread pool saturation or out-of-memory risk.

## Solution journey
### 1. No limits
Everything is allowed until the host collapses.

### 2. Fixed pools
Cap thread and connection counts.

### 3. Bulkheads
Separate critical payment work from non-critical background tasks.

### 4. Tuning and leak fixes
Reduce memory pressure and close resources properly.

### 5. Production-grade design
Protect the money path first, and move non-urgent work out of the synchronous request path.

## Tradeoffs
- Small pools reduce blast radius but lower peak throughput.
- Big pools increase concurrency but risk overload.
- Bulkheads improve isolation but add configuration overhead.
- Leak fixes are mandatory, but easy to miss without tests.

## Spring Boot lab
### Module
`capacity-service`

### Stages
1. `/resource-naive`
2. `/resource-pool`
3. `/resource-bulkhead`
4. `/resource-tuned`
5. `/resource-prod`

### Validation
Measure:
- thread count
- connection pool utilization
- heap usage
- open file descriptors
- rejected requests

### Final production-grade version
Keep synchronous payment endpoints tiny, isolate dependencies, and move heavy reporting or audit work out of the request path.
