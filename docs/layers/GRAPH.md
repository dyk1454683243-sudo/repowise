# Graph Intelligence

Most tools that draw a code graph will tell you `A calls B`. Very few will tell
you *how sure they are*, and none of the interesting questions can be answered
without that.

repowise builds a two-tier graph of your codebase, files and symbols, with no
model calls and no network. What makes it worth trusting is not its size. It is
that **every edge carries its own evidence**.

## How good is it, and how we know

Hand-graded edge-precision figures live in
[BENCHMARKS.md §7](../BENCHMARKS.md#7-edge-precision) as the single source of
truth:

| Tool | Figure |
|---|---|
| **repowise** | **240/280 = 85.7%** |
| **CodeGraph 1.5.0** | **164/280 = 58.6%** |

This page previously restated `229 of 270` / `154 of 270`, which drifted after a
language cell widened from 30 to 40 rows. Those counts are no longer duplicated
here so they cannot drift from BENCHMARKS.md again. Fixes #2352.

## See also

- [BENCHMARKS.md §7](../BENCHMARKS.md#7-edge-precision) — hand-graded edge precision
- [BENCHMARKS.md §8](../BENCHMARKS.md#8-the-same-question-against-an-answer-key-we-do-not-control) — compiler-oracle reading
- [LANGUAGE_SUPPORT.md](LANGUAGE_SUPPORT.md) — per-language resolution quality
