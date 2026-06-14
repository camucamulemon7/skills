---
name: reduce-rtl-power
description: Transforms synthesizable RTL to reduce dynamic or leakage power while preserving functional equivalence, cycle behavior, reset behavior, and module interfaces. Use when the user asks to reduce RTL power, switching activity, unnecessary toggles, clock-enable activity, operand activity, idle power, or equivalent Japanese requests such as "消費電力を減らす", "低電力化", "トグルを減らす", or "RTLを省電力にする".
---

# Reduce RTL Power

Use this skill to rewrite RTL for lower power without changing visible behavior. Logical equivalence is a hard precondition: do not apply a transformation unless the same observable cycle behavior can be preserved and justified.

## Workflow

1. Identify the target language, clock/reset style, module boundaries, enable protocol, and whether the user provided SAIF/VCD/FSDB, synthesis power reports, or switching reports.
2. Preserve public interfaces unless the user explicitly allows interface changes.
3. Preserve cycle behavior. If a transformation changes latency, throughput, handshake timing, reset observability, or any module-visible behavior, do not apply it under this skill; report it only as an out-of-scope architectural alternative when useful.
4. Load `references/rtl-power-patterns.md` when the task requires non-trivial low-power RTL transformations or when choosing among multiple power-reduction patterns.
5. Prioritize transformations that reduce unnecessary data-path toggles, idle updates, wide mux activity, repeated recomputation, and avoidable memory/register writes.
6. Keep the code synthesizable and compatible with the repository's existing RTL style.
7. Verify equivalence with existing simulation, formal checks, or focused tests. If no test infrastructure exists, state the remaining verification gap and provide a minimal check strategy.

## Edit Rules

- Avoid changing module ports, register-visible behavior, reset polarity, valid/ready semantics, or interrupt/status timing.
- Avoid adding clock gating cells directly unless the repository already has an equivalent clock-gating wrapper and its insertion preserves cycle behavior.
- Prefer clock-enable style RTL, data gating, operand isolation, local enables, write suppression, and mux input stabilization only when they preserve cycle behavior.
- Preserve attributes, pragmas, lint waivers, synthesis directives, CDC annotations, and low-power intent comments unless they are clearly obsolete after the rewrite.
- Keep naming deterministic and local to the transformed logic; do not rename unrelated signals.
- Treat X/unknown behavior, signedness, width extension, priority, and blocking/nonblocking scheduling as part of equivalence.

## Report Back

When finished, summarize:

- Which power source was addressed.
- Which equivalence assumptions were preserved.
- What verification was run.
- Any remaining power-analysis assumptions, such as missing activity files or tool-specific clock-gating inference.

## Example Requests

Requests that should use this skill include:

- "この RTL の無駄なトグルを減らして、等価性を保ったまま低電力化して。"
- "この datapath が idle 中も動いているので、レイテンシを変えずに消費電力を下げて。"
