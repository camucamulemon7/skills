# RTL Congestion Reduction Patterns

Use these patterns when reducing routing pressure in synthesizable Verilog, SystemVerilog, or VHDL. Apply only when logical equivalence, cycle behavior, reset semantics, and interface timing are preserved. If a transformation cannot meet that bar, treat it as out of scope for this skill.

## Contents

- Triage
- Cross-QoR Coordination
- Cycle-Preserving Transformations
- Out-of-Scope Architectural Changes
- Equivalence Checklist
- Verification Suggestions

## Triage

Look for high-fanout controls, wide centralized muxes, broadcast buses, long reductions, repeated remote indexing, and logic for independent lanes collapsed into one central block. Use fanout reports, congestion heatmaps, timing reports, and synthesis hierarchy reports when available.

## Cross-QoR Coordination

Treat congestion, timing, and power as coupled QoR axes. Before applying a transformation, record which QoR axis is the primary target and avoid later transformations that silently undo it.

- Replicating high-fanout control is intentionally opposite to sharing predecoded terms for timing. Use replication when physical fanout or spatial spread dominates; use shared predecode only when logic depth dominates and the resulting fanout remains acceptable.
- Partitioning wide muxes is the same structural family as timing-oriented hierarchical mux splitting. If one skill has already introduced mux hierarchy, refine the same hierarchy instead of applying a second independent split.
- Broadcast localization can increase register count, local clock-enable load, or power. Check the power skill's enable and write-suppression guidance if the duplicated state is active during idle cycles.

## Cycle-Preserving Transformations

### Replicate High-Fanout Control

Duplicate purely combinational decode near each consumer group when placement can keep each replica near its loads. Preserve required `dont_touch`, `keep`, or synthesis attributes if the tool would otherwise merge the replicas.

QoR side effect: this may improve congestion by reducing net spread, but it can increase area or dynamic power and can hurt timing if duplicated decode depth grows. Do not later collapse these replicas into one shared predecode unless timing evidence outweighs the congestion reason for replication.

### Partition Wide Muxes

Split a large mux by locality or lane group, then select within each group. Preserve `unique`/`priority` semantics and default behavior.

QoR side effect: this often also improves timing by reducing mux depth, but it can add local wires or area. If timing optimization follows, treat this as the existing mux hierarchy instead of creating another unrelated split.

### Replicate or Move Existing Broadcast Registers

Use existing architectural state in a more local form only when cycle behavior remains unchanged. Cycle-preserving localization means duplicating an already registered value or moving an existing register across equivalent combinational logic; it does not mean inserting a new pipeline register into the dataflow.

Safe cases:

- The original broadcast value is already registered, and equivalent local replicas can be updated in the same cycle under the same reset and enable conditions.
- An existing register can be legally moved across equivalent combinational logic without changing observable cycle timing, reset observability, or handshake behavior.
- The design already has per-lane or per-bank registers that can hold the broadcast value without adding a new architectural cycle.

Unsafe cases:

- Adding a new register on a source-to-consumer path generally changes latency and is not cycle-preserving.
- Reset initialization differs from the original path.
- Handshake or backpressure observes the extra storage.
- Debug, assertion, or status logic observes the original broadcast signal during invalid cycles.

### Balance Reductions

Replace linear OR/AND/XOR reductions with balanced trees when priority semantics are not required. Handle odd lane counts explicitly.

## Out-of-Scope Architectural Changes

Pipeline stages, staged crossbars, memory banking, distributed arbitration, request/response flops at hierarchy boundaries, or interface changes may reduce congestion, but they are not valid outputs of this skill unless they preserve the same observable cycle behavior.

## Equivalence Checklist

Check same observable cycle outputs, reset behavior, X handling, priority, signedness, width extension/truncation, blocking/nonblocking scheduling, clock-domain semantics, assertions, coverage, and lint intent.

## Verification Suggestions

Use existing simulations, formal equivalence, bounded property checks, focused randomized tests, synthesis lint, and CDC checks as appropriate. If no verification exists, provide a minimal self-checking testbench or formal wrapper when practical.
