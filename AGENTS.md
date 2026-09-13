# Software design depth doctrine - Agent instructions

> For OpenAI Codex and compatible agents

A design-review and design-authoring doctrine for fighting software complexity: deep modules,
information hiding, error handling, naming, comments. Apply this whenever you are designing a new
class/module/interface, reviewing or refactoring code, deciding how to decompose a system, naming
variables/methods, writing or reviewing comments, evaluating whether an abstraction is good, or
debating inheritance/TDD/agile/design-pattern tradeoffs. Also apply it to any diff that an AI/LLM
generated - "the model wrote it" is not a design decision, and it hides nothing on its own.

This doctrine is a personal synthesis built from weeks of deliberately internalizing the mental
models in John Ousterhout's _A Philosophy of Software Design_ until they became habitual judgment
rather than a checklist. Its central claim: **the biggest enemy of a software system is complexity,
and nearly every design decision is a decision about how to manage it.**

## The core idea: complexity

Complexity is not one big thing - it's the accumulation of small decisions:

```text
C = Σ (cp · tp)
```

Each part's complexity (`cp`), weighted by how much time developers spend on that part (`tp`). It
has three observable symptoms:

- **Change amplification** - a simple conceptual change requires code modifications in many places.
- **Cognitive load** - how much a developer must know to complete a task, even if that knowledge
  doesn't literally block them.
- **Unknown unknowns** - it isn't obvious what code needs to change, or even what to look at, to
  complete a task safely.

Complexity has exactly two root causes:

- **Dependencies** - a piece of code cannot be understood or changed in isolation.
- **Obscurity** - important information exists but isn't visible where you need it.

Every technique below is just a way to reduce dependencies, reduce obscurity, or both. If a proposed
fix doesn't do either, it's rearranging complexity, not addressing it.

Complexity is incremental - no single decision creates it, so no single decision fixes it either.
Favor an **investment mindset**: spend roughly 10-20% of your time on ongoing design improvement
instead of always taking the fastest tactical path. Skip that consistently and you get a "tactical
tornado" - a codebase where every fix is fast and every fix makes the next one slower.

## How to use this doctrine

**When designing something new:**

1. Ask what information this module/class/function should hide, and design the interface around
   hiding it.
2. Design it twice - sketch at least two substantially different approaches (interface and/or
   implementation) before committing.
3. Decide whether special cases and errors can be designed out of existence rather than handled.
4. Write the interface comment _before_ writing the implementation - if a comment is hard to write
   concisely, that's a design smell, not a writing problem.

**When reviewing or refactoring existing code:**

1. Walk the diff against the Red flags checklist below - each red flag names a specific, checkable
   symptom.
2. For each red flag found, trace it back to its root cause (dependency or obscurity) and propose a
   fix using the matching principle.
3. Don't just flag problems - recommend the deeper alternative (e.g., don't just say "this method is
   shallow," say what it should absorb or hide instead).

**When naming or commenting:** precision and consistency matter more than brevity - a name that
requires a comment to disambiguate is a naming failure, not a documentation gap.

**When evaluating a trend, pattern, or process** (OOP inheritance, TDD, agile, design patterns,
getters/setters, AI-generated code, performance work): ask "does this actually reduce dependencies
or obscurity, or does it just feel rigorous?"

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
| Shallow module | Interface isn't much simpler than the implementation behind it. |
| Information leakage | The same design decision (e.g., a file format, a data layout) is baked into more than one module. |
| Temporal decomposition | Code structure mirrors execution order ("first read, then parse, then write") rather than knowledge boundaries. |
| Overexposure | Using a common feature requires learning about rarely-used features first. |
| Pass-through method | A method does almost nothing but forward its arguments to another method with a similar signature. |
| Repetition | The same nontrivial code appears over and over. |
| Special-general mixture | Special-purpose logic is tangled into general-purpose code instead of being cleanly separated. |
| Conjoined methods | Two methods are so coupled you can't understand one without reading the other, even though nothing marks them as connected. |
| Comment repeats code | Everything the comment says is already obvious from the adjacent code. |
| Implementation documentation contaminates interface | An interface-level comment leaks implementation details the caller never needed. |
| Vague name | A name is so generic it conveys almost no information (`result`, `data`, `tmp`, `handle`). |
| Hard to pick name | Struggling to name something cleanly usually means the thing itself is not cleanly designed. |
| Hard to describe | If documenting something completely requires a long comment, the abstraction is probably wrong. |
| Nonobvious code | A reader can't tell what a piece of code does or why just by reading it. |

Each red flag is a symptom, not the disease - always ask "what dependency or obscurity is actually
causing this?" before proposing a fix. This applies identically whether the diff in front of you came
from a colleague or a language model: AI-assisted code doesn't add a third root cause, it just
multiplies the first two faster than review was built to absorb.

## Reference files

Full detail, examples, and code excerpts are split by theme in this repository - read only what's
relevant to the current task:

- `plugins/software-design-depth-doctrine/skills/software-design-depth-doctrine/references/complexity-and-strategy.md` -
  the complexity formula, tactical vs. strategic programming, the investment mindset.
- `plugins/software-design-depth-doctrine/skills/software-design-depth-doctrine/references/deep-modules-and-information-hiding.md` -
  deep vs. shallow modules, classitis, information hiding/leakage, general-purpose design, layering,
  pulling complexity downward.
- `plugins/software-design-depth-doctrine/skills/software-design-depth-doctrine/references/decomposition-and-errors.md` -
  when to merge vs. split classes/methods, defining errors out of existence, design-it-twice.
- `plugins/software-design-depth-doctrine/skills/software-design-depth-doctrine/references/comments-and-naming.md` -
  why comments matter, what makes a good comment, precise/consistent naming, writing comments first,
  modifying existing code, consistency.
- `plugins/software-design-depth-doctrine/skills/software-design-depth-doctrine/references/trends-and-performance.md` -
  OOP/inheritance, agile, unit tests, TDD, design patterns, getters/setters, designing for performance.
- `plugins/software-design-depth-doctrine/skills/software-design-depth-doctrine/references/applying-in-typed-fp-codebases.md` -
  how a subset of these principles show up concretely in typed/functional codebases (e.g. Scala).
  Load this only when reviewing or designing in such a codebase.

## Acknowledgment

Credit where due: the foundational framing (complexity as the root problem, deep modules,
information hiding, defining errors out of existence) originates with John Ousterhout's
_A Philosophy of Software Design_, referenced throughout as the intellectual source for the ideas
being applied.
