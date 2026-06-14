# RTL Congestion Reduction Patterns

Use these patterns when reducing routing pressure in synthesizable Verilog, SystemVerilog, or VHDL. Apply only when behavior, latency, and reset semantics are preserved unless the user explicitly approves an architectural change.

## Contents

- Triage
- Cycle-Preserving Transformations
- Optional Architectural Transformations
- Equivalence Checklist
- Verification Suggestions

## Triage

Look for high-fanout controls, wide centralized muxes, broadcast buses, long reductions, repeated remote indexing, and logic for independent lanes collapsed into one central block. Use fanout reports, congestion heatmaps, timing reports, and synthesis hierarchy reports when available.

## Cycle-Preserving Transformations

### Replicate High-Fanout Control

Duplicate purely combinational decode near each consumer group when placement can keep each replica near its loads. Preserve required `dont_touch`, `keep`, or synthesis attributes if the tool would otherwise merge the replicas.

### Partition Wide Muxes

Split a large mux by locality or lane group, then select within each group. Preserve `unique`/`priority` semantics and default behavior.

### Localize Broadcast Data

Register or replicate data at natural consumer boundaries only when cycle behavior remains unchanged. Do not add new storage if it changes latency, reset observability, handshake, or backpressure behavior.

### Balance Reductions

Replace linear OR/AND/XOR reductions with balanced trees when priority semantics are not required. Handle odd lane counts explicitly.

### Push Decode Toward Consumers

Replace one global decoded bus with smaller local decodes when each consumer needs only a subset. Keep centralized decode when it is part of a protocol boundary, assertion coverage, or shared formal abstraction.

## Optional Architectural Transformations

Ask before applying pipeline stages, staged crossbars, memory banking, distributed arbitration, request/response flops at hierarchy boundaries, or interface changes.

## Equivalence Checklist

Check same observable cycle outputs, reset behavior, X handling, priority, signedness, width extension/truncation, blocking/nonblocking scheduling, clock-domain semantics, assertions, coverage, and lint intent.

## Verification Suggestions

Use existing simulations, formal equivalence, bounded property checks, focused randomized tests, synthesis lint, and CDC checks as appropriate. If no verification exists, provide a minimal self-checking testbench or formal wrapper when practical.
