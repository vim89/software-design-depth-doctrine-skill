# Comments and naming

## Table of contents
- Four excuses for not writing comments (debunked)
- Why comments matter (tied to complexity causes)
- Comment categories
- Comment Repeats Code (red flag)
- Lower-level precision comments
- Higher-level intuition comments
- Interface documentation
- Implementation Documentation Contaminates Interface (red flag)
- Implementation comments (what/why, not how)
- Cross-module design decisions
- Choosing names: precision and consistency
- Vague Name / Hard to Pick Name (red flags)
- Write the comments first
- Hard to Describe (red flag)
- Modifying existing code
- Consistency

## Four excuses for not writing comments - and why they're wrong

1. **"Good code is self-documenting."** Some things genuinely are obvious from code, but a large and important category of information (why a design choice was made, what invariants hold, what a caller must not assume) cannot be expressed in code at all, no matter how well-named things are.
2. **"I don't have time to write comments."** This treats comments as a tax on top of "real" work rather than as part of the actual design process. Since comments capture design decisions, skipping them isn't just skipping documentation - it's skipping making those decisions explicit at all.
3. **"Comments get out of date and become misleading."** This is a maintenance discipline problem, not an argument against writing comments in the first place - the fix is to keep comments near the code they describe and update them as part of every change (see "Modifying existing code" below), not to abandon commenting.
4. **"All the comments I have seen are worthless."** Bad comments are common, but that's evidence of comments being written poorly (e.g., restating the code) - not evidence that good comments (ones capturing non-obvious information) lack value.

## Why comments matter - tied back to complexity's causes

Good comments directly attack both root causes of complexity: they reduce **cognitive load** (a reader doesn't have to reconstruct intent or hidden behavior from scratch) and eliminate **unknown unknowns** (a reader can discover constraints/assumptions that aren't visible in the code's structure at all). Comments are how you address **obscurity** and make **dependencies** explicit when the programming language itself can't express them.

## Comment categories

- **Interface comments** - describe what a class/method does and how to use it, without describing how it's implemented.
- **Data structure member comments** - document individual fields/variables, especially where their purpose or constraints aren't obvious from a type alone.
- **Implementation comments** - explain internal logic inside a method body: what a nontrivial block is doing and, crucially, *why*.
- **Cross-module documentation** - captures a design decision whose consequences span several modules, none of which is the "true home" of that decision (see `designNotes` technique below).

## Comment Repeats Code (red flag)

**Red flag: Comment Repeats Code** - a comment adds zero information beyond what's already obvious from the adjacent code. Examples cited: a Scala GUI scrollbar class comment that just restates the method name in prose, and a `normalizedResourceNames` comment that repeats the method signature without adding anything about behavior, assumptions, or edge cases. A good comment operates at a **different level of detail** than the code - either more precise (lower-level) or more abstract/intuitive (higher-level) than what the code itself shows.

## Lower-level precision comments

Sometimes a comment should be *more precise* than the code, nailing down exact units, boundary behavior, or edge-case semantics a reader can't infer just by reading the type or name. Examples: documenting exactly what an `offset` variable is measured in and from where; documenting exactly what `lineWidths` contains (per-line? cumulative? in characters or pixels?). Guidance for these comments: **think in terms of nouns, not verbs** - describe precisely what a value *is*, not just vaguely what it's *for*.

## Higher-level intuition comments

Sometimes a comment should instead convey the *intuition* or *purpose* behind a piece of code at a higher level of abstraction than the code itself shows - e.g., a RAMCloud RPC-handling comment that explains the overall purpose/strategy of a chunk of logic so a reader gets the "why" without tracing every line.

## Interface documentation

An interface comment should describe everything a *caller* needs to know to use the module correctly - and nothing about how it's implemented. Extended worked example: an `IndexLookup` class whose interface comment must clarify subtle behavioral questions (e.g., what does `isReady()` actually guarantee about state, and in what order must methods be called) - precisely the kind of informal-interface information that can't be expressed as a type signature.

**Red flag: Implementation Documentation Contaminates Interface** - an interface-level comment leaks details about *how* something is implemented that a caller never needed to know, unnecessarily increasing the caller's cognitive load and coupling the interface comment to implementation choices that might later change.

## Implementation comments

Implementation comments (inside a method body) should explain **what** a piece of code does and **why**, but generally not restate **how** the code performs a mechanical step the reader can already see. "Why" is the highest-value content here - e.g., "why does this call retry three times" is worth documenting; "this loop increments i" is not.

## Cross-module design decisions

Some design decisions have consequences that ripple across several modules, with no single obvious place to document them. Worked example: RAMCloud's `Status` enum, whose meaning and constraints matter across many otherwise-unrelated classes. Technique: a `designNotes` file (or equivalent central reference) that captures the decision once, with individual modules linking to it rather than each re-explaining the same cross-cutting decision independently (this doubles as a way to document subtle failure-handling logic, such as RAMCloud's handling of "zombie" servers).

## Choosing names

### The `block` variable bug story

A real anecdote: an ambiguous variable named `block` in Sprite OS took roughly six months to trace as the root cause of a subtle bug, precisely because the name did not make clear which of several related concepts ("block" as a disk block vs. as a memory block, etc.) it referred to. This motivates the chapter's central claim: imprecise names are not a cosmetic issue - they cause real, expensive bugs.

### Names should create an accurate image

A name's job is to let a reader form a correct mental picture of what the named thing is, without needing to read its full definition or usage sites.

### Names should be precise

Prefer specific, descriptive names over generic ones. Examples:
- `IndexletManager.getCount` - too generic; unclear what is being counted.
- Editor coordinate variables should be named for what they represent (not just terse `x`/`y` when more specific names would remove ambiguity).
- `blinkStatus` → renamed to `cursorVisible` - much more precise about what the boolean actually represents.
- `VOTED_FOR_SENTINEL_VALUE` → renamed to `NOT_YET_VOTED` - states the meaning directly instead of requiring the reader to infer intent from a sentinel-value convention.
- A generic `result` variable name, used because "it holds the result," conveys almost nothing about what that result actually *is*.

**Red flag: Vague name** - a name so imprecise it fails to convey useful information about the thing it names.

**Red flag: Hard to pick name** - difficulty finding a clean, precise name for something is itself a signal: it often means the underlying entity's purpose or boundaries are not well defined, not just that naming is hard in the abstract.

**Exception**: short-lived loop variables (`i`, `j`) are fine precisely because their scope is small enough that there's no real risk of ambiguity or of the reader losing track of what they mean.

**Over-specific naming pitfall**: a parameter originally named `selection` was found to be *too* narrow once the method's use expanded beyond selections specifically - renaming it to the more general `range` fixed the mismatch between the name's implied scope and its actual scope of use.

### Use names consistently

Adopt naming conventions and stick to them system-wide - e.g., a `srcFileBlock`/`dstFileBlock` naming pattern used consistently across a codebase, or an `i`/`j` nesting convention for nested loop indices - so that a reader's prior experience with the codebase transfers automatically to new code using the same names.

### A counter-opinion, and where the author still agrees

The official Scala style guide itself favors terseness in one specific, narrow case: generic type parameters are conventionally named with a single uppercase letter (`A`, `B`, `T`) rather than a descriptive name, reflecting a different philosophy about naming in that context. This doctrine disagrees with generalizing that terseness beyond type parameters, but concedes and adopts one specific piece of it: **the greater the distance between a name's declaration and its use, the longer and more descriptive the name should be** (short names are more defensible only when the reader will encounter the declaration and the use very close together).

## Write the comments first

**Delayed comments are bad comments** - comments written as an afterthought, after the code is "done," tend to be rushed, low-value restatements, because by that point the design decisions they should capture have already faded from the author's mind.

**Comments-first workflow**: write the class's interface comment first, then method signatures with their interface comments (bodies left empty), then instance-variable comments, and only then fill in the method bodies. This sequencing forces interface-level design thinking to happen *before* implementation, not as a rationalization after the fact.

**Comments as a design tool**: if writing a clean, short comment for something turns out to be difficult, treat that difficulty as a canary - it's usually revealing a real design problem, not just a writing problem.

**Red flag: Hard to describe** - if a variable or method needs a long comment to be fully and honestly documented, that complexity is a sign the abstraction itself needs rethinking, not just better prose.

**Early comments are fun, and cheap**: writing interface comments early turns them into a design exploration tool rather than a chore, and the actual time cost of comments is estimated at roughly 5% of total development time - a small price for the design clarity and future readability they buy.

## Modifying existing code

**Stay strategic when modifying, not just when creating** - the instinct to make "the smallest possible change" when fixing or extending existing code is itself a tactical-programming trap; the investment mindset applies just as much to modifications as to new code.

**Keep comments near the code they describe** - there's a real tradeoff between placing comments in header/interface files (visible to callers, but physically distant from the implementation) vs. inline in the implementation (closer to what's being described, but invisible to callers reading only the interface). A concrete pattern: use "phase" comments within an implementation to mark and describe distinct stages of a longer method.

**Comments belong in the code, not the commit log** - design rationale that only exists in a commit message is effectively invisible to anyone reading the code later; durable design knowledge belongs in the code itself.

**Avoid duplication when documenting** - reuse the `designNotes`-style reference technique instead of re-explaining another module's design decisions redundantly in multiple places; when documenting behavior defined by an external standard (e.g., the HTTP protocol), reference that external documentation rather than restating it.

**Check the diff before committing** - reviewing your own diff before committing is an explicit checkpoint for verifying comments were actually updated to match the code change, not left stale.

**Higher-level comments are easier to maintain** - comments pitched at a higher level of abstraction than the exact code details tend to survive small implementation changes without going stale, whereas comments that mirror implementation details precisely break every time those details shift.

## Consistency

**What consistency covers**: names, coding style, interface conventions, recurring design patterns, and invariants that hold across a codebase.

**How to ensure it**: document conventions explicitly, enforce them with automated tooling where possible (e.g., a script that checks line-termination conventions), reinforce them through code review, and - the "when in Rome" principle - follow a codebase's existing conventions rather than introducing your own preferred style into someone else's established code, unless you have a strong justification for changing the convention everywhere.

**Taking it too far**: consistency is not an excuse to force genuinely dissimilar things to look the same - things that are actually different should be done differently; forced consistency between unlike things creates its own obscurity.

## Code should be obvious

Obscurity is one of the two root causes of complexity, so this chapter is really about attacking obscurity directly through code-level presentation choices.

**What makes code more obvious**: consistent, deliberate use of whitespace/blank-line grouping to visually signal logical structure, and well-placed comments that surface information the code's structure alone can't convey.

**What makes code less obvious**:
- **Event-driven programming** - control flow is scattered across separate callback/handler registrations rather than following a single readable sequence, making the actual order of execution hard to reconstruct just by reading.
  **Red Flag: Nonobvious Code** - the behavior or meaning of a piece of code can't be understood easily from reading it.
- **Generic containers** - e.g., a Scala `(A, B)` tuple used generically loses the semantic meaning that a purpose-specific case class or named fields would have conveyed; ties back to Principle 14: **software should be designed for ease of reading, not ease of writing** - a generic container may be faster to write but costs every future reader.
- **Mismatched declaration/allocation types** - declaring a variable as a common supertype trait (`scala.collection.Seq`) while allocating a specific mutable implementation (`ArrayBuffer`) can obscure which concrete behaviors/guarantees are actually available.
- **Code that violates reader expectations** - e.g., a `RaftClient` example where threading behavior deviated from what a reader would reasonably assume from the surrounding code's conventions, creating a trap for anyone relying on those conventions.

**Three ways to keep code obvious**: reduce the amount of information a reader needs in the first place (via the deep-module and information-hiding techniques from earlier chapters), use knowledge the reader already has (established conventions, familiar idioms), and explicitly present necessary information through good names and comments where it can't be eliminated or assumed.
