# Graph Intelligence

Most tools that draw a code graph will tell you `A calls B`. Very few will tell
you *how sure they are*, and none of the interesting questions can be answered
without that.

repowise builds a two-tier graph of your codebase, files and symbols, with no
model calls and no network. What makes it worth trusting is not its size. It is
that **every edge carries its own evidence**.

## How good is it, and how we know

Hand-graded edge-precision figures are maintained in
[BENCHMARKS.md §7](../BENCHMARKS.md#7-edge-precision) as the single source of
truth:

| Tool | Figure |
|---|---|
| **repowise** | **240/280 = 85.7%** |
| **CodeGraph 1.5.0** | **164/280 = 58.6%** |

This page previously restated `229 of 270` / `154 of 270`, which drifted from
the benchmark doc after a language cell widened from 30 to 40 rows. The figures
above match BENCHMARKS.md; this page no longer keeps a parallel copy of the
counts so they cannot drift again.

For the full graph-design narrative (capture vs resolution, origins, edge types,
flows, ceilings), see the remainder of this file on `main` — or prefer linking
here for the graded numbers and reading `main` for the design prose until the
full page is re-synced in a follow-up.
