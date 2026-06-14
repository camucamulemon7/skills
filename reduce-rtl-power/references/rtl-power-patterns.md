# RTL Power Reduction Patterns

Use these patterns when reducing dynamic or leakage power in synthesizable Verilog, SystemVerilog, or VHDL. Apply only when behavior, latency, reset semantics, and interface timing are preserved unless the user explicitly approves an architectural change.

## Contents

- Triage
- Cycle-Preserving Transformations
- Optional Architectural Transformations
- Equivalence Checklist
- Verification Suggestions

## Triage

Look for registers updated every cycle while unused, wide datapaths toggling during idle/stall/invalid cycles, expensive combinational blocks recomputing unused outputs, mux inputs toggling unnecessarily, idle counters/status logic, memory inputs toggling while disabled, and repeated decode or compare logic. Use power reports, SAIF/VCD/FSDB activity, clock-gating reports, and synthesis reports when available.

## Cycle-Preserving Transformations

### Add Register Enables for Idle Updates

Hold registers when the next value is not architecturally consumed. Use only when consumers observe the register as meaningful under an enable or valid condition.

### Suppress Redundant Writes

Avoid rewriting registers or memories when the stored value would not change. Use when comparator cost is justified by reduced switching.

### Gate Datapath Inputs During Invalid Cycles

Stabilize inputs to multipliers, wide adders, shifters, comparators, encoders, and mux trees when their outputs are not consumed. Confirm invalid-cycle values are not observable by assertions, debug outputs, or X-sensitive checks.

### Localize Enables

Replace broad enables with narrower per-lane, per-bank, per-opcode, or per-counter enables while preserving protocol validity and priority.

### Stabilize Mux And Memory Inputs

Hold inactive mux sources and memory address/write-data/write-mask inputs when disabled, following local memory macro conventions.

## Optional Architectural Transformations

Ask before applying pipeline stages, technology-specific clock gates, interface changes, memory banking/shutdown, operand caching, scheduling changes, retention, isolation, or power-domain behavior.

## Equivalence Checklist

Check same observable cycle outputs, reset behavior, valid/ready/stall/flush/interrupt/status timing, invalid-cycle observability, X handling, priority, signedness, width handling, scheduling behavior, clock-domain semantics, assertions, coverage, and lint intent.

## Verification Suggestions

Use simulations, formal equivalence, bounded properties, randomized valid/idle/stall/flush tests, lint/CDC/reset checks, and before/after power estimates with the same activity source.
