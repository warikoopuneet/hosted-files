# Network partitions and CAP tradeoffs

## Problem
A network split can isolate part of the system. In payments, the hard decision is whether to keep accepting writes or preserve consistency by refusing them.

## Naive implementation
Assume the network always works.

## Failure scenario
Break connectivity between nodes and observe split-brain risk or unavailability.

## Solution journey
### 1. Ignore partitions
Unsafe.

### 2. Quorum write model
Only a majority can commit.

### 3. Read-only minority
Partially isolated nodes stop taking writes.

### 4. Eventual merge
Accept writes on both sides and reconcile later.

### 5. Production-grade design
Critical payment paths usually prefer consistency over availability.

## Tradeoffs
- Quorum protects correctness but reduces availability.
- AP-style acceptance keeps the service up but can create conflicts.
- Reconciliation pushes complexity into the back office.
- Read-only fallback is safer than split-brain writes.

## Spring Boot lab
### Module
`replicated-ledger-service`

### Stages
1. `/write-naive`
2. `/write-quorum`
3. `/write-readonly-fallback`
4. `/write-eventual`
5. `/write-prod`

### Validation
Measure:
- successful writes during partition
- rejected writes during partition
- conflict count after healing
- recovery time

### Final production-grade version
Use quorum-based writes for the ledger and graceful degradation for non-critical read paths.
