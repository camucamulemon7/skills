---
name: reduce-rtl-routing-congestion
description: Transforms synthesizable RTL to reduce routing congestion while preserving functional equivalence, cycle behavior, timing intent, reset behavior, and module interfaces. Use when the user asks to reduce wiring/routing congestion, high fanout, long interconnect, crossbar/mux pressure, floorplan-unfriendly RTL, placement/routing difficulty, or equivalent Japanese requests such as "配線混雑を減らす", "混雑を緩和", "配線を軽く", or "P&RしやすいRTL".
---

# Reduce RTL Routing Congestion

Use this skill to rewrite RTL for lower routing pressure without changing visible behavior. Logical equivalence is a hard precondition: do not apply a transformation unless the same observable cycle behavior can be preserved and justified.

## Workflow

1. Identify the target language, tool assumptions, module boundaries, clock domains, reset style, and whether the user provided synthesis/P&R reports.
2. Preserve public interfaces unless the user explicitly allows interface changes.
3. Preserve cycle behavior. If a transformation changes latency, handshake timing, reset observability, or any module-visible behavior, do not apply it under this skill; report it only as an out-of-scope architectural alternative when useful.
4. Load `references/rtl-patterns.md` when the task requires non-trivial RTL transformations or when choosing among multiple congestion-reduction patterns.
5. Prioritize transformations that reduce high fanout, wide long-distance muxes, broadcast buses, centralized decode, global enables, and repeated cross-module control nets.
6. Keep the code synthesizable and compatible with the repository's existing RTL style.
7. Verify equivalence with existing simulation, formal checks, or focused tests. If no test infrastructure exists, state the remaining verification gap and provide a minimal check strategy.

## Edit Rules

- Avoid changing module ports, register-visible behavior, reset polarity, or valid/ready semantics.
- Do not add pipeline stages or new architectural storage.
- Prefer local registered control/data replicas, partitioned muxing, replicated decoders, and balanced reduction trees only when they preserve cycle behavior.
- Preserve attributes, pragmas, lint waivers, synthesis directives, and clock-domain annotations unless they are clearly obsolete after the rewrite.
- Keep naming deterministic and local to the transformed logic; do not rename unrelated signals.
- Use generate blocks or small helper functions only when they improve structure without hiding hardware cost.

## Report Back

When finished, summarize:

- Which congestion source was addressed.
- Which equivalence assumptions were preserved.
- What verification was run.
- Any remaining physical-design assumptions, such as expected placement locality or fanout limits.

## Example Requests

Requests that should use this skill include:

- "この SystemVerilog の高ファンアウトな enable を、等価性を保ったまま配線混雑が下がる形に直して。"
- "この wide mux が P&R で混雑を作っているので、レイテンシを変えずに RTL を改善して。"
