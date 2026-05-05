# Clock skew, ordering, and hybrid clocks

## Problem
Different machines disagree about time. In payments, that can break ordering, reconciliation, and stale-read protection.

## Naive implementation
Use local wall clock timestamps as if they were globally reliable.

## Failure scenario
Run two nodes with different clock offsets and compare event order.

## Solution journey
### 1. Local wall clock only
Simple, but unreliable across nodes.

### 2. Clock synchronization
Use system time sync as a baseline.

### 3. Logical ordering
Use sequence numbers or logical clocks when real time is not enough.

### 4. Hybrid logical clock
Combine wall time and monotonic sequencing.

### 5. Production-grade design
Use timestamps carefully, and never trust them as the sole source of correctness for money movement.

## Tradeoffs
- Wall clock is easy but unsafe for ordering.
- Logical clocks are safer but less human-friendly.
- Hybrid clocks are practical, but still require design discipline.
- Strong time guarantees usually need extra infrastructure.

## Spring Boot lab
### Module
`event-order-service`

### Stages
1. `/time-naive`
2. `/time-synced`
3. `/time-logical`
4. `/time-hlc`
5. `/time-prod`

### Validation
Check:
- timestamp monotonicity
- event ordering
- read-your-writes behavior

### Final production-grade version
Use logical ordering for correctness and wall-clock timestamps only for display and auditing.
