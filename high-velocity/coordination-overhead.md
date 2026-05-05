# Coordination overhead and cross-service transaction cost

## Problem
A payment often spans authorization, fraud check, ledger write, and notification. If every step is synchronous, the transaction becomes slow and fragile.

## Naive implementation
A coordinator calls several services in sequence and waits for each one.

## Failure scenario
Inject 50 ms to 200 ms network delay in one downstream service.
The whole payment slows down because the coordinator is blocked.

## Solution journey
### 1. Naive synchronous orchestration
Simple, but every hop adds latency.

### 2. Reduce hops
Combine logic where safe and remove unnecessary service calls.

### 3. Async workflow
Return accepted, continue processing in the background, and persist state transitions.

### 4. Saga
Each service performs a local transaction and publishes the next step.
Use compensation when a later step fails.

### 5. Production-grade design
Use event-driven processing and a durable workflow or ledger backbone.
That avoids keeping client threads blocked across the whole chain.

## Tradeoffs
- Synchronous flow is easiest to reason about but slowest.
- Async flow improves latency but creates eventual consistency.
- Saga preserves progress but needs compensation logic.
- Fewer service calls help latency but may increase service coupling.

## Spring Boot lab
### Module
`payment-orchestrator`

### Stages
1. `/pay-sync`
2. `/pay-batched`
3. `/pay-async`
4. `/pay-saga`
5. `/pay-workflow`

### Validation
Measure:
- end-to-end latency
- number of network calls per payment
- compensation count
- workflow completion time

### Docker
Run:
- orchestrator
- account service
- fraud service
- notification service
- optional queue broker

### Final production-grade version
Use a durable workflow or event stream, with local ACID commits per service and explicit compensation steps where needed.
