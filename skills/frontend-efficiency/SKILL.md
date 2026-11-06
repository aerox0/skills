---
name: frontend-efficiency
description: Apply framework-independent frontend efficiency defaults when designing, implementing, refactoring, or reviewing interfaces, client state, collection processing, rendering, data fetching, and CSS effects. Use to prevent repeated work, unnecessary updates, request waterfalls, main-thread blocking, and excessive rendering cost while preserving visual quality. No mandatory profiling or benchmarking workflow.
---

# Frontend Efficiency

Keep interactions responsive. Make work proportional to what changed and what the user needs to see. Apply these defaults within the requested scope and the existing framework's update model.

- Avoid accidental O(n²) processing. Use maps or sets for repeated lookups when worthwhile; compute shared results once and hoist invariant work. Preserve ordering, duplicate handling, and equality semantics.
- Keep updates local. Subscribe to only the state needed, derive values instead of synchronizing redundant state, and avoid unrelated rerenders.
- Preserve stable item identities. Memoize expensive repeated work when useful, not every value or callback.
- Bound rendering for large collections with pagination or virtualization, preserving accessibility, keyboard navigation, and focus.
- Avoid layout thrashing: group DOM reads and writes. Keep high-frequency event handlers lightweight and coalesce visual updates when appropriate.
- Keep long computations off interaction paths; chunk work or use workers when justified. Async functions alone do not move CPU work off-thread.
- Avoid request waterfalls and duplicate fetches. Bound concurrency, fetch only needed data, and cancel or ignore stale requests.
- Load optional features and offscreen assets on demand. Prioritize critical content, size media appropriately, and reserve space to prevent layout shifts.
- Clean up listeners, subscriptions, timers, and background work. Bound caches and retained data; avoid unnecessary continuous animations.
- Prefer simple code for small, bounded workloads. Preserve correctness and usability; measure uncertain trade-offs rather than adding speculative optimization infrastructure.

## CSS rendering cost

- Use backdrop blur and translucent surfaces intentionally when they improve the design. Favor small elements, a single navigation bar, or a limited number of prominent cards.
- Keep the total visible blurred area and number of overlapping blur layers modest. Avoid applying blur indiscriminately across dense grids or lists.
- Account for what moves behind a blurred surface: even a static navigation bar can require ongoing rendering work while content scrolls beneath it. Prefer static blur settings over animating blur strength.
- Offscreen-heavy layouts can justify richer effects, but ensure offscreen content actually skips rendering rather than assuming it is free.
- If blur causes responsiveness issues, reduce its radius, affected area, or simultaneous instances before replacing it with a simpler translucent or solid fill. Preserve the visual intent where practical.
- Keep large, layered shadows and other filters deliberate. If costly, simplify the shadow, reduce the affected area, or use a border or pre-rendered static effect where it preserves the design.
- Prefer animating `transform` and `opacity` over layout properties such as `width`, `height`, `top`, and `left`. Avoid animating shadows, filters, or large gradients when a simpler effect communicates the same state. Compositing is not a guarantee of low cost.
- Avoid `transition: all`; name only the properties intended to animate.
- Use `will-change` sparingly and temporarily. Do not force GPU layers on every element; excessive layers consume memory and can hurt performance.
- Consider `content-visibility: auto` for substantial offscreen sections, with appropriate intrinsic sizing. Use CSS containment only where its layout and clipping behavior are compatible with the design.
- Treat these as cost-sensitive patterns, not blanket bans. Static effects on small areas are often fine; prioritize reducing affected area, repaint frequency, and simultaneous effects. Respect reduced-motion preferences.

## Verification

Check relevant observable behavior, especially rapid input, stale responses, large collections, navigation, cleanup, and keyboard focus. Use focused checks appropriate to the change. Do not require a profiling or benchmarking project for routine improvements; measure uncertain or substantial trade-offs and support numeric speed claims with evidence.
