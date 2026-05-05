# Hot keys, skew, and shard hotspots

## Problem
A small number of accounts, merchants, or invoice IDs receive a disproportionate share of traffic. One shard becomes overloaded while the rest sit idle.

## Naive implementation
Use one table, one partitioning rule, or one leader for everything.

## Failure scenario
Send all traffic for a popular merchant to the same key range and watch latency spike.

## Solution journey
### 1. Naive single hot shard
Simple, but brittle.

### 2. Better key design
Increase key cardinality and avoid sequential hot keys.

### 3. Salting or hashing
Spread writes across multiple physical partitions.

### 4. Explicit sharding
Route payments by account or merchant hash to different nodes.

### 5. Production-grade design
Use automatic hotspot detection, adaptive partition movement, and careful cache design.

## Tradeoffs
- Salting complicates reads.
- Sharding requires router logic.
- Rebalancing can be operationally hard.
- Rate limiting protects the system but can reject valid traffic.

## Spring Boot lab
### Module
`merchant-routing-service`

### Stages
1. `/pay-hotspot`
2. `/pay-salted`
3. `/pay-sharded`
4. `/pay-rebalanced`
5. `/pay-prod`

### Validation
Track:
- per-shard request rate
- CPU per node
- p99 latency by merchant
- queue depth per partition

### Final production-grade version
Use adaptive routing plus a ledger design that avoids concentrating every write for one hot entity on a single partition.
