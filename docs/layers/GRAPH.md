# Graph Intelligence

Most tools that draw a code graph will tell you `A calls B`. Very few will tell
you *how sure they are*, and none of the interesting questions can be answered
without that.

repowise builds a two-tier graph of your codebase, files and symbols, with no
model calls and no network. What makes it worth trusting is not its size. It is
that **every edge carries its own evidence**.

<p>
  <img src="https://img.shields.io/badge/17-edge_types-3178C6?style=flat-square&labelColor=0A0A0A" alt="17 edge types" />
  <img src="https://img.shields.io/badge/29-resolution_origins-059669?style=flat-square&labelColor=0A0A0A" alt="29 resolution origins" />
  <img src="https://img.shields.io/badge/26-languages-F59520?style=flat-square&labelColor=0A0A0A" alt="26 languages" />
  <img src="https://img.shields.io/badge/22-framework_detectors-7F52FF?style=flat-square&labelColor=0A0A0A" alt="22 framework detectors" />
  <img src="https://img.shields.io/badge/0-LLM_calls-1E293B?style=flat-square&labelColor=0A0A0A" alt="zero LLM calls" />
  <img src="https://img.shields.io/badge/compiler_graded-7_of_7_cells_undominated-DC2626?style=flat-square&labelColor=0A0A0A" alt="no tool is both more precise and more complete, in 7 of 7 compiler-graded cells" />
</p>

**Contents:** [The problem with a plain arrow](#the-problem-with-a-plain-arrow) ·
[Two stages, and they fail differently](#two-stages-and-they-fail-differently) ·
[How good is it, and how we know](#how-good-is-it-and-how-we-know) ·
[What is in the graph](#what-is-in-the-graph) ·
[Every edge says how it got there](#every-edge-says-how-it-got-there) ·
[Typing the receiver](#typing-the-receiver) ·
[Seventeen edge types](#seventeen-edge-types-because-calls-was-doing-too-many-jobs) ·
[Flows that say why they stopped](#flows-that-say-why-they-stopped) ·
[What the graph powers](#what-the-graph-powers) ·
[Seeing it yourself](#seeing-it-yourself) ·
[Honest ceilings](#honest-ceilings)

---

## The problem with a plain arrow

Consider one line of Python:

```python
user.save()
```

To draw an edge, a tool has to answer "what is `user`?". There are three ways to
do it, and they are not equally good:

1. **Give up.** Emit nothing. The edge is missing, so a dead-code pass now
   thinks `save` is unused and offers to delete it.
2. **Guess.** Find every method named `save` in the repo and pick one. If your
   codebase has `User.save`, `Draft.save` and `Session.save`, you have a two in
   three chance of drawing an arrow to the wrong file.
3. **Work out what `user` is**, then resolve `save` on that type.

Most graphs do (2) and present the result identically to an edge they were
certain about. That is the actual problem. A wrong arrow is worse than a missing
one, because a missing arrow looks like missing information and a wrong arrow
looks like an answer.

repowise does (3) where it can, falls back to (2) where it must, and **labels
which one happened, every time**.

---

## Two stages, and they fail differently

An edge is two claims made by two different pieces of machinery, and the reason
to keep them apart is that they break in opposite directions.

**Stage one: capture.** Before anything can be resolved, the parser has to
notice that a call was written at all. That is a tree-sitter query per language,
listing the source shapes that count as a call site:

```scheme
; queries/go.scm -- Method call: obj.Method(args)
(call_expression
  function: (selector_expression
    operand: (identifier) @call.receiver
    field: (field_identifier) @call.target
  )
  arguments: (argument_list) @call.arguments
) @call.site
```

There is one of those files per language, and a shape that is not in it is a
call the graph will never contain. Go alone lists a plain call, a method call, a
package-qualified call, a chained call and a function passed as an argument, and
that last one is captured deliberately as a *reference* rather than a call,
because passing a handler is not invoking it.

A shape no query matches is invisible. Not low confidence, not unresolved:
**absent**. Nothing downstream can recover it, because nothing downstream knows
the call was there.

**Stage two: resolution.** Given a captured site, work out what the name points
at. `repo.save(draft)` hands you the name `save` and a receiver spelled `repo`,
and the job is to turn that into one declaration in one file. This is where the
29 origins below live, and it is the `user.save()` problem from the section
above.

| | fails when | costs you | how you find out |
|---|---|---|---|
| **capture** | nobody wrote a query for that call shape | **recall.** the edge does not exist | nothing internal can tell you, so it takes an outside answer key |
| **resolution** | the receiver cannot be typed, or the name is ambiguous | **precision** if it guesses, **recall** if it declines | the origin on the edge names the strategy that answered, so a wrong class is found and fixed once rather than per call site |

That asymmetry sets the whole design. A missed capture is silent, so it is
measured against a compiler rather than against ourselves. A bad resolution is
loud, so every edge is stamped with the strategy that produced it and nothing is
allowed to launder a guess into a fact.

**And at the bottom of the ladder repowise declines rather than guesses.** That
is a choice with a price, paid in the recall column below, and it is the right
way round: a missing arrow looks like missing information, a wrong arrow looks
like an answer.

---

## How good is it, and how we know

Two readings of the same question, and we graded only one of them.

### The one we did not grade

On Go the answer key is the **Go team's own RTA call graph** from
`golang.org/x/tools`, computed over the fully type-checked program. On TypeScript
it is the **`tsc` checker's own resolution** of every call site. We wrote
neither, we can tune neither, and anyone with the toolchain can regenerate both.

Seven cells, five repositories, **five tools**, 37,853 oracle edges. Of the call
edges we emit, the share the compiler confirms runs **0.943 to 0.992** per cell.

That number on its own is not the claim, because precision on its own has a cheap
way to win: draw one edge you are certain of and you score 1.000. Two of the five
tools do a version of exactly that. One scores 0.997 on cobra, the highest figure
in the whole experiment, from a graph holding **17% of the calls in the
repository**. Another takes both gitleaks cells from a graph that
finds 89% of the calls where we find 95%. Meanwhile the tool with the best recall emits, on the largest
repository measured, more than a third of its edges as calls the compiler says do
not exist.

**Recall alone is gameable in the other direction and precision alone in this
one, so the claim is the pair:**

> In all seven cells, **no tool that recovers as much of the call graph as we do
> gets more of it right.**

It names no threshold, so it cannot be tuned, and a new competitor can only break
it. Two were added after it was written and it held in all seven cells.

The weaker readings, so nobody has to infer them: most precise outright in one
cell, tied in one more, beaten in five by tools drawing much smaller graphs. And
against the two tools the experiment started with, most precise in seven of
seven, which is the narrower claim it should always be labelled as.

**Two languages, and only two.** C#, Java, Kotlin and C++ each need a toolchain
installed and a working build per repository, and nobody has done that here.
Python, Ruby and PHP can never have an oracle at all, because what a call
resolves to can change at runtime. That is a fact about those languages rather
than a gap in the harness, and it is why the hand-graded reading below is
permanent rather than a stopgap.

[The cells, the method and the graded pre-registration](../BENCHMARKS.md#8-the-same-question-against-an-answer-key-we-do-not-control)

### The one we did

Nine languages, mostly 30 call edges per language per tool (Java is 40), every
row opened in its own file with its imports and enclosing scope, then the target
declaration opened too. **240 of 280 correct for us, 164 of 280 for CodeGraph
1.5.0**, intervals disjoint. Four of the nine cells separate and five are ties,
reported as ties.

Read our own number the other way round: **roughly fourteen percent of our call
edges are wrong**, concentrated in java, rust and cpp. That is the figure to plan
against, and it is a floor rather than a best case, because every resolver change
since the earliest rows were graded only removes wrong edges.

**The two readings agree.** On Go the hand grade says 96.7% for us and the
compiler says 97.6%, over roughly 1,600 edges rather than 30 rows. A person
reading source and a type checker landing within about a point of each other is
the strongest available evidence that the hand-graded half is accurate rather
than self-serving, and it is the result here we care about most.

[The nine cells, and all 540 graded rows with the reason each was given](../BENCHMARKS.md#7-edge-precision)
