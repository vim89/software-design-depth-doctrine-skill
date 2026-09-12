# Complexity and strategy

## Table of contents
- Programming as a creative, incremental activity
- The definition and formula of complexity
- Three symptoms of complexity
- Two root causes of complexity
- Complexity is incremental - zero-tolerance philosophy
- Tactical vs. strategic programming
- The investment mindset

## Programming as a creative, incremental activity

Software design is not a one-shot activity you finish before writing code. It's ongoing: every time you touch a system, you are also making a design decision, whether consciously or not. You cannot fully visualize a complex system at the outset of a project, so the best designs emerge incrementally, refined as you learn more. The book's two goals are (1) to describe the nature of complexity - the ways it manifests, and how it's caused - and (2) to present techniques you can use during design to minimize complexity.

## Definition and formula of complexity

Complexity is anything related to the structure of a software system that makes it hard to understand and modify. It is not a single global property; it is local and cumulative:

```
C = Σ (cp · tp)
```

where `cp` is the complexity of a particular part `p`, and `tp` is the fraction of time developers spend working on that part. A system can have one horribly complex method that never matters much because nobody touches it, or many mildly complex parts that dominate overall difficulty because they're touched constantly. This is why complexity is inherently something you evaluate part-by-part, not just as a single verdict on "the codebase."

## Three symptoms of complexity

- **Change amplification**: a seemingly simple change requires modifications in many different places. Example: changing a color used throughout a website's banners requires updates everywhere that color is hardcoded, instead of one shared definition.
- **Cognitive load**: how much a developer needs to know to complete a task - even API surface they must be aware of but don't literally need to invoke correctly still adds cognitive load.
- **Unknown unknowns**: the worst of the three. It isn't obvious what code must be modified, or what information is even relevant, to complete a task correctly. Unlike the first two symptoms, this one isn't even discoverable through study - you may not know you got it wrong until it breaks.

## Two root causes

- **Dependencies**: a piece of code cannot be understood, changed, or verified in isolation from other code. Some dependencies are inherent to a system and can't be eliminated, only exposed cleanly (e.g., a method signature is a dependency between caller and callee, unavoidable but manageable).
- **Obscurity**: important information is not obvious. Obscurity often results from dependencies that aren't clearly documented, or from inconsistent naming/structure that hides intent.

## Complexity is incremental

No single decision creates unmanageable complexity - it accumulates in small increments, one shortcut at a time. This has an important corollary: since no single change is the smoking gun, developers systematically underestimate the damage of "just this one hack." The correct response is a zero-tolerance philosophy: sweat the small stuff. Treat small complexity increases as seriously as large ones, because they are the actual mechanism by which systems rot.

## Tactical vs. strategic programming

- **Tactical programming**: the goal is to get something working - a feature shipped, a bug fixed - as fast as possible, with design as an afterthought (if it's thought about at all). This is what most pressure (deadlines, "just ship it") pushes developers toward.
- **The Tactical Tornado**: an archetype of a programmer who ships code faster than anyone else, is beloved by management for velocity, but leaves behind a wake of complexity that slows every other developer down. Their code "works" but accretes shortcuts, near-duplicate logic, and undocumented assumptions.
- **Strategic programming**: the primary goal is not "working code" but a great design, where working code is a byproduct. Strategic programmers accept spending extra time now to save much more time later.

Working code isn't enough - a system that works but is hard to extend or maintain has already failed at the actual job, even if it currently passes its tests.

## The investment mindset

Make continual small investments to improve system design, rather than treating design improvement as a separate, deferrable phase. A commonly cited target is roughly 10-20% of total development time invested in design (naming things well, writing good comments, refactoring rough edges, considering a second design alternative) - not because that's a hard rule, but because a small, steady investment compounds, while zero investment compounds in the opposite direction.

Two real-world reference points illustrate both sides:
- **Facebook's early "Move Fast and Break Things"** motto embodied pure tactical programming; the company later explicitly reversed course to "Move Fast With Solid Infrastructure" once accumulated complexity from the tactical era started slowing everyone down.
- **Google and VMware** are cited as counter-examples where strategic, long-horizon investment in system design paid off in velocity over time, rather than eroding it.

The core danger described is a slippery slope: each individual decision to defer cleanup seems locally reasonable ("we'll fix it after this deadline"), but there is never a natural point where the debt gets repaid on its own - deferred cleanup has to be a deliberate choice, made repeatedly, or it simply never happens.
