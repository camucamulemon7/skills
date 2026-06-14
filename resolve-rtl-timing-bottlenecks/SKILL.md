---
name: resolve-rtl-timing-bottlenecks
description: Transforms synthesizable RTL to resolve timing bottlenecks while preserving logical equivalence, cycle behavior, reset behavior, and module interfaces. Use when the user asks to fix critical paths, improve Fmax, reduce combinational delay, address setup timing violations, shorten long mux/decode/arithmetic paths, or equivalent Japanese requests such as "タイミングボトルネックを解消", "クリティカルパスを短く", "Fmaxを上げる", or "setup違反を直す".
---

# Resolve RTL Timing Bottlenecks

Use this skill to rewrite RTL for better timing closure without changing visible behavior.

## Workflow

1. Identify the target language, clock/reset style, module boundaries, timing target, failing path type, and whether the user provided STA reports, synthesis timing reports, or path schematics.
2. Preserve public interfaces unless the user explicitly allows interface changes.
3. Preserve cycle behavior by default. If a useful transformation changes latency, throughput, handshake timing, or reset observability, present it as an optional architectural change instead of applying it silently.
4. Load `references/rtl-timing-patterns.md` when the task requires non-trivial timing transformations or when choosing among multiple critical-path reduction patterns.
5. Prioritize transformations that reduce logic depth, simplify priority chains, split wide mux/decode paths, balance reductions, precompute shared terms, and make the design more retiming-friendly without altering behavior.
6. Keep the code synthesizable and compatible with the repository's existing RTL style.
7. Verify equivalence with existing simulation, formal checks, or focused tests. If no test infrastructure exists, state the remaining verification gap and provide a minimal check strategy.

## Edit Rules

- Avoid changing module ports, register-visible behavior, reset polarity, valid/ready semantics, exception timing, or status timing unless explicitly requested.
- Avoid adding pipeline stages or moving architectural registers across observable boundaries unless the user accepts latency or timing-contract changes.
- Prefer balanced combinational structure, localized muxing, predecoded control, one-hot or encoded conversion where equivalent, common-subexpression extraction, and retiming-friendly RTL.
- Preserve attributes, pragmas, lint waivers, synthesis directives, CDC annotations, and timing exceptions unless they are clearly obsolete after the rewrite.
- Keep naming deterministic and local to the transformed logic; do not rename unrelated signals.
- Treat X/unknown behavior, signedness, width extension, priority, and blocking/nonblocking scheduling as part of equivalence.

## Report Back

When finished, summarize:

- Which timing bottleneck was addressed.
- Which equivalence assumptions were preserved.
- What verification was run.
- Any remaining timing-closure assumptions, such as missing STA reports, synthesis retiming settings, or physical placement effects.

## Example Requests

Requests that should use this skill include:

- "この SystemVerilog の critical path を、論理等価性を保ったまま短くして。"
- "この priority mux が setup 違反になっているので、レイテンシを変えずに RTL を改善して。"
