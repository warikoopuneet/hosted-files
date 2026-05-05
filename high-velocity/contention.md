# Contention, race conditions, idempotency, and locking

## Problem
Two or more payment requests touch the same account, invoice, or ledger row at the same time. In payments, this can cause double debit, lost updates, or incorrect final balances.

## Naive implementation
A Spring Boot controller directly reads a balance, subtracts an amount, and writes it back.

## Failure scenario
Run 50 to 200 concurrent requests against the same account.
The classic bug appears:
- request A reads balance 100
- request B reads balance 100
- both subtract 30
- final balance becomes 70 instead of 40

## Solution journey
### 1. Naive approach
Fast, simple, and wrong under contention.

### 2. Idempotency key
Store request IDs and return the prior result for duplicates.
This protects retry storms and client resend loops.

### 3. Pessimistic locking
Lock the row before update.
Correct, but concurrent callers wait and tail latency rises.

### 4. Optimistic locking
Add a version column and retry on conflict.
Higher throughput than strict locking, but conflict handling becomes application logic.

### 5. Serializable isolation
Let the database enforce the strongest guarantee.
Very safe for money movement, but expensive under heavy write contention.

### 6. Sharding
Move from one hot account table to partitioned account ranges.
This reduces contention per shard and scales the write path.

## Production pattern
Payment systems typically combine:
- idempotency keys on public APIs
- database transactions for the ledger
- optimistic version checks
- retries with bounded backoff
- a double-entry ledger model

## Tradeoffs
- Idempotency fixes duplicate submissions, not concurrent distinct writes.
- Pessimistic locking fixes correctness but lowers concurrency.
- Optimistic locking improves throughput but increases retries.
- Serializable isolation is safest, but most costly.
- Sharding scales best, but adds routing and rebalancing complexity.

## Spring Boot lab
### Module
`payment-account-service`

### Stages
1. `/debit-naive`
2. `/debit-idempotent`
3. `/debit-locked`
4. `/debit-optimistic`
5. `/debit-sharded`

### Validation
Assert:
- no negative balances unless intentionally allowed
- duplicate requests do not create duplicate debits
- final sum across all accounts matches expected value

### Docker
Run:
- app container
- postgres container
- load-test container

### Final production-grade version
Use a payment ledger service with:
- idempotency key table
- `@Transactional`
- `@Version`
- retry policy with jitter
- partitioned account routing
- audit event log
