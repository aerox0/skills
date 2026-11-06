---
name: data-and-behavior
description: Apply language-independent guidance for clear data models and behavior when designing, implementing, refactoring, or reviewing software. Use for repeated conditional logic, configuration, business-rule variations, lifecycle workflows, state transitions, and separation of decisions from side effects. Prefer shared logic with explicit data where it simplifies the problem, without forcing state machines or generic rule engines.
---

# Data-driven Behavior

When multiple cases follow the same structure, prefer representing their
differences as data and processing them with shared code.

- Use tables, mappings, or configuration for repeated values and rules
  when this makes behavior easier to understand and change.
- Keep algorithms, complex decisions, and side effects in explicit code.
- For lifecycle-dependent behavior, make valid states and transitions
  explicit. Use a transition table when it simplifies the implementation.
- Validate externally supplied configuration at its boundary and handle
  missing or unsupported cases explicitly.
- Keep rules in one authoritative place and test meaningful outcomes,
  including invalid cases.

Apply this where a clear pattern exists. Prefer the simplest readable
implementation; do not introduce configuration systems, rule engines,
or state machines without a concrete need. Follow existing project
conventions and avoid unrelated refactoring.
