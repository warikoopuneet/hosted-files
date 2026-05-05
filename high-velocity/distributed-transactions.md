# Distributed transactions, sagas, and ACID

## Problem
A payment may need to debit one account and credit another account in a different service or shard. If one side commits and the other fails, money disappears or gets duplicated.

## Naive implementation
Call two independent services and hope both succeed.

## Failure scenario
Stop the second service after the first has already committed.
You now have partial state.

## Solution journey
### 1. Independent local transactions
Wrong for money movement across boundaries.

### 2. Two-phase commit
Prepare both sides, then commit only if every participant is ready.
Correct, but blocking and slower.

### 3. Saga
Each step is local and has a compensation path.
More available, but not truly atomic across the whole workflow.

### 4. Event sourcing
Persist immutable events and rebuild state from the log.
This helps auditability and recovery.

### 5. Production-grade design
For the strictest payment paths, use a system that gives serializable transactions across shards.
For broader workflows, use sagas plus an immutable ledger.

## Tradeoffs
- 2PC gives strong correctness but can block under failure.
- Saga improves availability but requires compensation and careful design.
- Event sourcing improves auditability but increases complexity in reads and replays.
- Distributed SQL simplifies ACID at the cost of infrastructure complexity.

## Spring Boot lab
### Module
`transfer-service`

### Stages
1. `/transfer-naive`
2. `/transfer-2pc-sim`
3. `/transfer-saga`
4. `/transfer-events`
5. `/transfer-prod`

### Validation
Check:
- total balance conservation
- compensation correctness
- no partial commit after simulated crash
- replay produces the same ledger state

### Final production-grade version
Use a ledger service that writes immutable events, then materialize balances from those events or from a serializable transactional store.
