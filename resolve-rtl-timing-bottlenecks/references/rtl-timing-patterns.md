# RTL Timing Bottleneck Patterns

Use these patterns when reducing critical-path delay in synthesizable Verilog, SystemVerilog, or VHDL. Apply only when logical equivalence, cycle behavior, reset semantics, and interface timing are preserved unless the user explicitly approves an architectural change.

## Contents

- Triage
- Cycle-Preserving Transformations
- Optional Architectural Transformations
- Equivalence Checklist
- Verification Suggestions

## Triage

Look for long priority chains, wide muxes, deep arithmetic paths, decode plus mux plus arithmetic in one cycle, long reductions, population counts, leading-zero counts, repeated array indexing, variable part-selects, artificial combinational dependencies, and STA paths dominated by logic depth. Use STA reports, synthesis timing reports, path schematics, and constraints when available.

## Cycle-Preserving Transformations

### Balance Reductions

Replace linear reductions with balanced trees when associativity makes it equivalent. Do not use for priority logic unless priority order is preserved.

### Split Wide Muxes Hierarchically

Turn one large mux into smaller local muxes followed by a final select. Preserve `unique`, `priority`, and default semantics.

### Preserve Priority While Reducing Depth

For priority encoders or first-match logic, compute independent match groups and combine group winners in the original priority order.

### Predecode Shared Terms

Compute common compares or decodes once, then reuse them when it shortens repeated logic cones without creating problematic high fanout.

### Simplify Add-Compare-Select Chains

Move invariant terms out of the critical path and avoid recomputing equivalent arithmetic. Explicitly preserve widths, overflow, and signedness.

### Make Retiming Legal And Obvious

Use clear `always_ff` register boundaries and `always_comb` logic so synthesis can retime if enabled. Do not change asynchronous reset behavior just to enable retiming unless approved.

## Optional Architectural Transformations

Ask before applying pipeline stages, moving registers across module boundaries, handshake timing changes, skid/elastic buffers, multicycle timing, multi-cycle exact operations, memory architecture changes, datapath width changes, or reset style changes.

## Equivalence Checklist

Check same observable cycle outputs, reset behavior, valid/ready/stall/flush/exception/interrupt/status timing, X handling, priority, signedness, width extension/truncation/overflow, scheduling behavior, clock-domain semantics, assertions, coverage, and lint intent.

## Verification Suggestions

Use simulations, formal equivalence, bounded properties, focused randomized tests around overlap/boundary/priority cases, lint/CDC checks, and synthesis or STA comparison with the same constraints when available.
