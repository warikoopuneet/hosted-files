# Tail latency and hedged requests

## Problem
Most requests may be fast, but a few stragglers break the SLA. In payments, those tail events can cause timeouts, retries, and user-visible failures.

## Naive implementation
Use one code path, one dependency, and one timeout for everything.

## Failure scenario
Inject random 100 ms to 500 ms stalls in a small percentage of requests.

## Solution journey
### 1. Naive single request
No protection from slow outliers.

### 2. Timeouts
Fail fast rather than waiting forever.

### 3. Retries with backoff
Recover from transient delays, but only if the work is safe to retry.

### 4. Hedged requests
Send a backup request after a short delay and use the first successful response.

### 5. Production-grade design
Combine fast timeouts, safe retries, bulkheads, and replica-aware routing.

## Tradeoffs
- Timeouts improve responsiveness but increase error count.
- Retries help transient failures but can amplify load.
- Hedging lowers p99 latency but increases total traffic.
- Replica selection adds routing complexity.

## Spring Boot lab
### Module
`payment-query-service`

### Stages
1. `/balance-naive`
2. `/balance-timeout`
3. `/balance-retry`
4. `/balance-hedged`
5. `/balance-prod`

### Validation
Measure:
- p50/p95/p99 latency
- total request count
- retry count
- hedge win rate

### Final production-grade version
Use fast local response rules, safe fallbacks, and hedged reads only where duplication is acceptable and idempotent.
