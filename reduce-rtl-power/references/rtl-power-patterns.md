# RTL Power Reduction Patterns

Use these patterns when reducing dynamic or leakage power in synthesizable Verilog, SystemVerilog, or VHDL. Apply only when logical equivalence, cycle behavior, reset semantics, and interface timing are preserved. If a transformation cannot meet that bar, treat it as out of scope for this skill.

## Contents

- Triage
- Cross-QoR Coordination
- Cycle-Preserving Transformations
- Out-of-Scope Architectural Changes
- Equivalence Checklist
- Verification Suggestions

## Triage

Look for registers updated every cycle while unused, wide datapaths toggling during idle/stall/invalid cycles, expensive combinational blocks recomputing unused outputs, mux inputs toggling unnecessarily, idle counters/status logic, memory inputs toggling while disabled, and repeated decode or compare logic. Use power reports, SAIF/VCD/FSDB activity, clock-gating reports, and synthesis reports when available.

## Cross-QoR Coordination

Treat power, congestion, and timing as coupled QoR axes. Before applying a transformation, record which QoR axis is the primary target and avoid later transformations that silently undo it.

- Clock-enable and write-suppression logic can add enable fanout and routing pressure. Check the congestion skill when a new enable would drive many distant registers.
- Operand isolation can improve power but add muxes or logic depth. Check the timing skill when the isolated block is on a critical path.
- Sharing predecoded terms may reduce repeated switching, but it can create a high-fanout control net. Use local replicas when congestion evidence dominates.

## Cycle-Preserving Transformations

### Add Register Enables for Idle Updates

Hold registers when the next value is not architecturally consumed. Use only when consumers observe the register as meaningful under an enable or valid condition.

QoR side effect: this can reduce dynamic power, but the enable itself may become high fanout and can hurt congestion or timing if it is not localized.

### Suppress Redundant Writes

Avoid rewriting registers or memories when the stored value would not change. Use when comparator cost is justified by reduced switching.

QoR side effect: the compare logic can lengthen the write-enable path. Do not apply it on a critical path unless timing remains equivalent and acceptable.

### Gate Datapath Inputs During Invalid Cycles

Stabilize inputs to multipliers, wide adders, shifters, comparators, encoders, and mux trees when their outputs are not consumed. Confirm invalid-cycle values are not observable by assertions, debug outputs, or X-sensitive checks.

### Localize Enables

Replace broad enables with narrower per-lane, per-bank, per-opcode, or per-counter enables while preserving protocol validity and priority.

### Stabilize Mux And Memory Inputs

Hold inactive mux sources and memory address/write-data/write-mask inputs when disabled, following local memory macro conventions.

## Out-of-Scope Architectural Changes

Pipeline stages, technology-specific clock gates, interface changes, memory banking/shutdown, operand caching, scheduling changes, retention, isolation, or power-domain behavior may reduce power, but they are not valid outputs of this skill unless they preserve the same observable cycle behavior.

## Equivalence Checklist

Check same observable cycle outputs, reset behavior, valid/ready/stall/flush/interrupt/status timing, invalid-cycle observability, X handling, priority, signedness, width handling, scheduling behavior, clock-domain semantics, assertions, coverage, and lint intent.

## Verification Suggestions

Use simulations, formal equivalence, bounded properties, randomized valid/idle/stall/flush tests, lint/CDC/reset checks, and before/after power estimates with the same activity source.
