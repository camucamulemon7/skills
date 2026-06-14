# RTL Timing Bottleneck Patterns

Use these patterns when reducing critical-path delay in synthesizable Verilog, SystemVerilog, or VHDL. Apply only when logical equivalence, cycle behavior, reset semantics, and interface timing are preserved. If a transformation cannot meet that bar, treat it as out of scope for this skill.

## Contents

- Triage
- Cross-QoR Coordination
- Cycle-Preserving Transformations
- Out-of-Scope Architectural Changes
- Equivalence Checklist
- Verification Suggestions

## Triage

Look for long priority chains, wide muxes, deep arithmetic paths, decode plus mux plus arithmetic in one cycle, long reductions, population counts, leading-zero counts, repeated array indexing, variable part-selects, artificial combinational dependencies, and STA paths dominated by logic depth. Use STA reports, synthesis timing reports, path schematics, and constraints when available.

## Cross-QoR Coordination

Treat timing, congestion, and power as coupled QoR axes. Before applying a transformation, record which QoR axis is the primary target and avoid later transformations that silently undo it.

- Sharing predecoded terms is intentionally opposite to congestion-oriented high-fanout replication. Share terms only when logic depth dominates and fanout remains acceptable.
- Hierarchical mux splitting is the same structural family as congestion-oriented mux partitioning. Refine an existing mux hierarchy instead of introducing a second independent split.
- Balancing or extracting logic can increase switching activity. Check the power skill when a timing fix makes idle logic toggle more often.

## Cycle-Preserving Transformations

### Balance Reductions

Replace linear reductions with balanced trees when associativity makes it equivalent. Do not use for priority logic unless priority order is preserved.

### Split Wide Muxes Hierarchically

Turn one large mux into smaller local muxes followed by a final select. Preserve `unique`, `priority`, and default semantics.

QoR side effect: this often also improves congestion when grouping follows physical locality, but it can add local wires or area. If congestion optimization already partitioned the mux, reuse that hierarchy.

### Preserve Priority While Reducing Depth

For priority encoders or first-match logic, compute independent match groups and combine group winners in the original priority order.

### Predecode Shared Terms

Compute common compares or decodes once, then reuse them when it shortens repeated logic cones without creating problematic high fanout.

QoR side effect: shared predecode can reduce logic depth but may create a high-fanout control net and worsen routing congestion. Do not collapse replicated local decodes unless timing evidence outweighs the congestion reason for replication.

### Simplify Add-Compare-Select Chains

Move invariant terms out of the critical path and avoid recomputing equivalent arithmetic. Explicitly preserve widths, overflow, and signedness.

### Make Retiming Legal And Obvious

Use clear `always_ff` register boundaries and `always_comb` logic so synthesis can retime if enabled. Do not change asynchronous reset behavior just to enable retiming.

## Out-of-Scope Architectural Changes

Pipeline stages, moving registers across module boundaries, handshake timing changes, skid/elastic buffers, multicycle timing, multi-cycle exact operations, memory architecture changes, datapath width changes, or reset style changes may improve timing, but they are not valid outputs of this skill unless they preserve the same observable cycle behavior.

## Equivalence Checklist

Check same observable cycle outputs, reset behavior, valid/ready/stall/flush/exception/interrupt/status timing, X handling, priority, signedness, width extension/truncation/overflow, scheduling behavior, clock-domain semantics, assertions, coverage, and lint intent.

## Verification Suggestions

Use simulations, formal equivalence, bounded properties, focused randomized tests around overlap/boundary/priority cases, lint/CDC checks, and synthesis or STA comparison with the same constraints when available.
