---
name: software-design-depth-doctrine
description: >
  A personal design-review and design-authoring doctrine for fighting software
  complexity: deep modules, information hiding, error handling, naming, comments.
  Use for designing a new class/module/interface; reviewing or refactoring code;
  deciding how to decompose a system; naming variables/methods; writing or reviewing
  comments; evaluating whether an abstraction is good; or debating inheritance/TDD/agile/design-pattern
  tradeoffs. Trigger on phrases like "is this a good design," "how
  should I structure this," "does this class do too much," "how should I comment
  this," or any code review requesting design-quality feedback.
---

# Software design depth doctrine

A design-review and design-authoring doctrine built from several weeks spent deliberately studying, internalizing, and applying the mental models for managing software complexity - deep modules, information hiding, error handling, naming, comments - until they became habitual judgment rather than a checklist. As a courtesy: this doctrine owes its foundational framing to John Ousterhout's *A Philosophy of Software Design*, whose central claim anchors everything here: **the biggest enemy of a software system is complexity, and nearly every design decision is a decision about how to manage it.** Use this skill to make that judgment explicit - as a lens for designing new code and as a checklist for reviewing existing code.

## The core idea: complexity

Complexity is not one big thing - it's the accumulation of small decisions (`C = Σ cp · tp`: each part's complexity weighted by how much time developers spend on that part). It has three observable symptoms:

- **Change amplification** - a simple conceptual change requires code modifications in many places.
- **Cognitive load** - how much a developer must know to complete a task, even if that knowledge doesn't literally block them.
- **Unknown unknowns** - it isn't obvious what code needs to change, or even what to look at, to complete a task safely.

Complexity has two root causes: **dependencies** (a piece of code cannot be understood/changed in isolation) and **obscurity** (important information is not obvious). Every technique in this skill is just a way to reduce dependencies, reduce obscurity, or both.

Complexity is incremental - no single decision creates it, so no single decision fixes it either. This is why the skill favors an **investment mindset**: spend a bit of extra time (roughly 10-20%) on ongoing design improvement rather than always taking the fastest tactical path. Tactical, feature-first shortcuts compound into a "tactical tornado" of unmanageable complexity.

## How to use this skill

**When designing something new:**
1. Ask what information this module/class/function should hide, and design the interface around hiding it (see `references/deep-modules-and-information-hiding.md`).
2. Design it twice - sketch at least two substantially different approaches (interface and/or implementation) before committing (see `references/decomposition-and-errors.md`).
3. Decide whether special cases and errors can be designed out of existence rather than handled (same reference file).
4. Write the interface comments *before* writing the implementation - if a comment is hard to write concisely, that's a design smell, not a writing problem (see `references/comments-and-naming.md`).

**When reviewing or refactoring existing code:**
1. Walk the code against the **Red Flags Checklist** below - each red flag names a specific, checkable symptom.
2. For each red flag found, trace it back to its root cause (dependency or obscurity) and propose a fix using the matching principle.
3. Don't just flag problems - recommend the deeper alternative (e.g., don't just say "this method is shallow," say what it should absorb or hide instead).

**When naming or commenting:**
See `references/comments-and-naming.md` for the precision/consistency rules for names and the four categories of comments.

**When evaluating a trend, pattern, or process (OOP inheritance, TDD, agile, design patterns, getters/setters, performance work):**
See `references/trends-and-performance.md` - evaluate any new methodology by asking "does this actually reduce dependencies or obscurity, or does it just feel rigorous?"

## The 15 design principles (quick reference)

1. Complexity is incremental: sweat the small stuff.
2. Working code isn't enough - it must also be well-designed.
3. Make continual small investments to improve system design.
4. Modules should be deep.
5. Interfaces should make the most common usage as simple as possible.
6. A simple interface matters more than a simple implementation.
7. General-purpose modules are deeper.
8. Separate general-purpose code from special-purpose code.
9. Different layers should have different abstractions.
10. Pull complexity downward (into the implementation, away from callers).
11. Define errors (and special cases) out of existence.
12. Design it twice.
13. Comments should describe things that are not obvious from the code.
14. Software should be designed for ease of reading, not ease of writing.
15. The increments of software development should be abstractions, not features.

## Red flags checklist (use this during review)

| Red flag | Symptom to look for |
|---|---|
| **Shallow module** | Interface isn't much simpler than the implementation behind it. |
| **Information leakage** | The same design decision (e.g., a file format, a data layout) is baked into more than one module. |
| **Temporal decomposition** | Code structure mirrors execution order ("first read, then parse, then write") rather than knowledge boundaries. |
| **Overexposure** | Using a common feature requires learning about rarely-used features first. |
| **Pass-Through method** | A method does almost nothing but forward its arguments to another method with a similar signature. |
| **Repetition** | The same nontrivial code appears over and over. |
| **Special-General mixture** | Special-purpose logic is tangled into general-purpose code instead of being cleanly separated. |
| **Conjoined methods** | Two methods are so coupled you can't understand one without reading the other, even though nothing marks them as connected. |
| **Comment repeats code** | Everything the comment says is already obvious from the adjacent code. |
| **Implementation documentation contaminates interface** | An interface-level comment leaks implementation details the caller never needed. |
| **Vague name** | A name is so generic it conveys almost no information (`result`, `data`, `tmp`, `handle`). |
| **Hard to pick name** | Struggling to name something cleanly usually means the thing itself is not cleanly designed. |
| **Hard to describe** | If documenting something completely requires a long comment, the abstraction is probably wrong. |
| **Nonobvious code** | A reader can't tell what a piece of code does or why just by reading it. |

Each red flag is a symptom, not the disease - always ask "what dependency or obscurity is actually causing this?" before proposing a fix.

## Reference files

Full detail, examples, and code excerpts are split by theme - load only what's relevant to the current task:

- `references/complexity-and-strategy.md` - the complexity formula, tactical vs. strategic programming, the investment mindset.
- `references/deep-modules-and-information-hiding.md` - deep vs. shallow modules, classitis, information hiding/leakage, general-purpose design, layering, pulling complexity downward.
- `references/decomposition-and-errors.md` - when to merge vs. split classes/methods, defining errors out of existence, design-it-twice.
- `references/comments-and-naming.md` - why comments matter, what makes a good comment, precise/consistent naming, writing comments first, modifying existing code, consistency.
- `references/trends-and-performance.md` - OOP/inheritance, agile, unit tests, TDD, design patterns, getters/setters, designing for performance.
- `references/applying-in-typed-fp-codebases.md` - how a small set of these principles (design it twice, define errors out of existence, the design-patterns caution, clarity over cleverness, small general-purpose interfaces) show up concretely in typed/functional codebases (e.g. Scala). Load this only when reviewing or designing in such a codebase - it adds no new doctrine, only concrete translations.

## Output format: tips & tricks

Before defaulting to another paragraph of prose, consider escalating the output format - each
step below tends to carry more information per unit of reader effort than the one before it:

1. **Controlled-language prose** - for plain-text explanations, write (or ask for) ASD-STE100
   style: short sentences, one idea per sentence, a restricted, unambiguous vocabulary. It was
   built for aerospace maintenance manuals, but the constraints produce explanations that are
   far more readable than typical free-form prose. The full spec is quite stringent - asking for
   "80% of the way to ASD-STE100" is often the better trade-off.
2. **Diagrams** - before writing another explanatory paragraph, ask whether a diagram (sequence,
   state machine, call graph, architecture) would convey the same structure faster. Diagrams are
   often easier to parse than the equivalent prose, especially for anything with more than two
   moving parts or a nontrivial order of operations.
3. **Interactive web pages** - for a comparison, a walkthrough, or a dashboard, consider asking
   for output "in HTML" instead of a static document. An interactive page lets the reader explore
   instead of forcing a single linear read.
4. **Explainer videos** - for a genuinely complex topic, consider a fully custom, narrated
   explainer video (e.g. "create a 3b1b-style video explainer on X"). This needs an API key for
   narration (or a local-compute alternative) but is now realistic to ask for.

The underlying shift: as LLMs absorb more of the legwork autonomously, more of the work rises
into oversight and understanding rather than production - and because intelligence and code are
increasingly abundant, it's worth asking for large, custom, *discardable* artifacts (a one-off web
app, a one-off video) that would never have been worth building by hand.

## Acknowledgment

This doctrine is a personal synthesis, not a transcription - it reflects weeks of deliberately practicing these mental models until they became instinct. Credit where due: the foundational framing (complexity as the root problem, deep modules, information hiding, defining errors out of existence) originates with John Ousterhout's *A Philosophy of Software Design*, referenced throughout as the intellectual source for the ideas being applied.
