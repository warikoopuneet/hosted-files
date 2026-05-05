# Scalability ceilings and Amdahl-style limits

## Problem
Adding more nodes does not always increase throughput. The serial part of the system becomes the bottleneck.

## Naive implementation
Scale out every component and expect linear improvement.

## Failure scenario
Measure throughput as you add instances and see it flatten.

## Solution journey
### 1. Add more instances
Good first move, but not sufficient.

### 2. Find the serial path
Identify locks, coordinators, hot partitions, or synchronous dependencies.

### 3. Parallelize safely
Break work into independent pieces.

### 4. Move non-critical work off-path
Use async processing for reporting, fraud analytics, and audit enrichment.

### 5. Production-grade design
Design the critical payment path to be as short and parallel as possible, then use asynchronous systems for the rest.

## Tradeoffs
- More instances increase cost.
- Parallelism improves throughput only when tasks are independent.
- Off-path processing improves speed but adds eventual consistency.
- Simplification of the hot path is often the biggest win.

## Spring Boot lab
### Module
`system-scale-service`

### Stages
1. `/scale-naive`
2. `/scale-profiled`
3. `/scale-parallel`
4. `/scale-async`
5. `/scale-prod`

### Validation
Measure:
- throughput vs instance count
- serial fraction
- queueing delay
- saturation point

### Final production-grade version
Keep the critical money path compact and push everything else to async pipelines and batch jobs.
