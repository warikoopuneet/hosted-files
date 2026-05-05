# Leader election churn and replicated systems

## Problem
A replicated system can spend too much time electing new leaders. During churn, writes stall and payment traffic becomes unstable.

## Naive implementation
Use the default leader settings and hope elections are rare.

## Failure scenario
Pause or slow the leader and observe repeated elections.

## Solution journey
### 1. Default leader election
Simple, but sensitive to pauses.

### 2. Longer timeouts
Avoid false elections caused by transient stalls.

### 3. Better heartbeat stability
Reduce noise and avoid aggressive failover.

### 4. Sharding leadership
Split the workload so one leader does not own everything.

### 5. Production-grade design
Use stable quorum settings, careful timeout tuning, and partitioned responsibility.

## Tradeoffs
- Fast failover improves resilience but may cause churn.
- Slow failover reduces churn but delays recovery.
- Sharding reduces blast radius but increases operational complexity.

## Spring Boot lab
### Module
`replica-controller-service`

### Stages
1. `/leader-naive`
2. `/leader-failover`
3. `/leader-tuned`
4. `/leader-sharded`
5. `/leader-prod`

### Validation
Track:
- leader changes
- write unavailability
- election duration
- recovery time

### Final production-grade version
Use well-tuned consensus groups and keep them small enough to recover quickly.
