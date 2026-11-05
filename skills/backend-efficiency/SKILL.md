---
name: backend-efficiency
description: Apply language-independent backend efficiency patterns while designing, implementing, refactoring, or reviewing collection processing, APIs, services, jobs, and data access. Use for algorithmic complexity, data-structure selection, repeated computation, N+1 I/O, batching, caching, incremental processing, and bounded resource use. Favor efficient defaults without requiring a profiling or benchmarking workflow.
---

# Backend Efficiency

Make work proportional to the data needed and the changes made. Prefer efficient algorithms and data flow before micro-optimizations. Preserve correctness, clear code, and the existing architecture unless a concrete constraint justifies a change.

## Apply during the task

1. Read the relevant implementation and callers. Identify input sizes, call frequency, data access, and existing limits. Use documented constraints; label unknown scale assumptions rather than inventing targets.
2. Trace total work across the request or job, including helpers, loops, retries, and downstream calls. Look for repeated scans, recomputation, round trips, and unbounded accumulation.
3. Choose the simplest approach that fits the workload. Apply the relevant defaults below within the requested scope; do not turn a local change into a system-wide optimization project.
4. Verify behavior with focused existing checks or meaningful tests. For reviews, report findings without editing unless requested.
5. Explain material changes in complexity, I/O count, or resource bounds and their trade-offs. Distinguish analytical improvements from measured speedups.

## Algorithms and data structures

- Avoid accidental quadratic work: repeated searches, membership checks, joins, or copies inside growing loops. Inspect helper costs as well as explicit nested loops.
- Build an index once for repeated lookups when the construction and memory cost are worthwhile. Use hash maps/sets for average constant-time lookup; preserve duplicate, ordering, and equality semantics.
- Select structures for the required operations: lookup, ordering, range search, deduplication, FIFO processing, or top-k selection. Use efficient queue operations rather than repeatedly shifting large arrays.
- Use a single pass for min/max or aggregation when sufficient. Consider a bounded heap for top-k rather than sorting everything; do not assume every problem can be solved in O(n).
- Hoist invariant parsing, compilation, allocation, and computation out of loops. Avoid repeatedly copying a growing collection or string.
- Retain simple scans for small, bounded collections or one-off lookups when an index would add needless overhead. Do not merge readable linear passes solely to reduce the loop count.

## Do shared work once

- Compute shared inputs and derived results once per operation, then pass them to consumers. Avoid hidden re-fetching or recomputation in per-item helpers.
- Prefer updating the affected portion of state over rebuilding all derived state after each small change, when consistency can be maintained simply.
- Deduplicate simultaneous identical expensive work when safe. Scope deduplication by authorization, tenant, inputs, and freshness requirements; clean up entries on failure or cancellation.
- Reuse expensive clients and connection pools according to their lifecycle and concurrency guarantees. Keep mutable request state isolated.
- Introduce cross-request caching only for a concrete reuse opportunity. Define key scope, size limits, freshness, invalidation, and miss behavior; do not add a new caching service by default.

## Data access and I/O

- Avoid N+1 queries and per-item remote calls. Fetch related data in batches or joins and index the results for local matching. Chunk batches to respect parameter and payload limits.
- Request only necessary fields and rows. Filter and aggregate near the data source when that reduces transfer and total work.
- Design database indexes around actual filters, joins, and ordering. Account for selectivity, existing indexes, and write/storage cost rather than indexing every field.
- Paginate, chunk, or stream large results. Preserve ordering and consistency requirements; avoid collecting a stream back into an unbounded list.
- Overlap independent I/O with bounded concurrency. Preserve dependency order, transaction semantics, and downstream capacity; do not parallelize every item without a limit.
- Keep transactions and lock hold times short. Avoid unrelated network calls while holding database locks.

## Bound resource use

- Set limits appropriate to the workload for inputs, payloads, batch sizes, concurrent work, queues, retries, caches, and retained results. Reuse existing limits where available.
- Enforce timeouts and propagate cancellation. Retry only eligible failures with a bounded budget and backoff; account for idempotency and retries in other layers.
- Apply backpressure or reject excess work when producers can outrun consumers. Moving work to a background queue does not remove its cost.
- Keep substantial CPU work off latency-sensitive request/event loops using the runtime's appropriate execution mechanism. Do not introduce workers for trivial operations.
- Release connections, streams, locks, and temporary resources on success, failure, and cancellation.

## Quick reasoning examples

| Pattern | Consider instead | Check before changing |
| --- | --- | --- |
| Scan m users for each of n orders | Build a user index once: average O(n + m) time, O(m) extra space | Duplicate IDs, missing matches, ordering |
| Recalculate the same configuration for every record | Calculate once per batch and reuse | Whether configuration can change during the batch |
| Load each item's related records separately | Bounded batched fetch plus local matching | Authorization, query limits, snapshot consistency |
| Rebuild all aggregates after one update | Update affected aggregates incrementally | Deletes, retries, concurrent writes, recovery |
| Start one promise/task per record in a huge input | Bounded worker pool with backpressure | Failure propagation, cancellation, output order |

## Verification and reporting

- Check relevant edge cases: empty input, duplicates, missing keys, ordering, partial failures, and concurrent updates. Preserve authorization and transactional behavior.
- Use focused behavior tests where warranted; do not add tests that merely assert the chosen implementation or flaky wall-clock thresholds.
- Do not require benchmarks or profiling for routine structural improvements. Measure when the benefit is uncertain, trade-offs are substantial, or a concrete latency/throughput claim needs evidence.
- Report only material findings or decisions. State the old and new work pattern, the applicable scale assumption, and any added memory or consistency cost. Do not claim a numeric speedup from Big-O alone.
