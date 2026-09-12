# Applying these principles in typed/FP codebases (e.g. Scala)

This file does not introduce new doctrine. It maps a small number of practices seen in
typed-functional codebases back to specific principles from this doctrine, so the connection
stays verifiable rather than a vague "good engineering" gesture. Every bullet below cites the
exact principle it instantiates. If a practice from a language- or framework-specific style
guide (SOLID mapping, CI review templates, git/gh workflow, effect libraries, diagram
policies) can't be traced to a specific principle, it does not belong here - it belongs in
that language's own skill, not in this one.

## Table of contents
- Design it twice, formalized
- Making invalid states unrepresentable
- The design-patterns caution, applied to FP machinery
- Clarity over cleverness
- Keep the interface small, general-purpose

## Design it twice, formalized

When a style guide requires "a simpler alternative and why it fails" before adopting a
pattern, or requires listing tradeoffs with explicit downsides before finalizing a system
design, that's "design it twice" made into a checklist item rather than a
one-time suggestion. The point stands: the value isn't the alternative itself, it's
that having a second option to compare against is what reveals the first design's hidden
weaknesses. Apply this the same way regardless of language - require a genuinely different
second option, not a cosmetic variation of the first.

## Making invalid states unrepresentable

"Define errors out of existence" argues for redesigning semantics so an error
condition simply can't arise (Tcl's `unset` no-op, Unix allowing deletion of open files).
In a typed language, the equivalent technique is modeling domain state with sum types (ADTs)
or distinct value types so that a category of bug (wrong state passed to wrong place, a
primitive mixed up with another primitive of the same underlying type) becomes a compile
error instead of a runtime failure to catch and handle. This is the same design goal -
eliminate the error case rather than write code to handle it - expressed through a
language's type system instead of through API semantics. It does not replace the guidance
on masking/aggregating errors that genuinely can occur; use it only for errors that
can be designed away entirely.

## The design-patterns caution, applied to FP machinery

This doctrine is explicit: design patterns are broadly useful because they encode
well-validated solutions, but "the main risk is over-application" - forcing a problem into a
familiar pattern when a custom, purpose-fit approach would be cleaner. The same caution
applies directly to FP-specific machinery (tagless final, free monads, optics, phantom
types, compile-time specialization): each is a pattern, and each should be justified by a
concrete problem it solves, not adopted because it's available or looks rigorous. A style
guide that gates these behind explicit questions ("what concrete invalid state exists
today," "can a teammate maintain this in six months," "is runtime validation actually
insufficient") is applying the same yardstick, not inventing a new one. Treat "more
type-level machinery" with the same suspicion this doctrine gives "more classes" (classitis)
- it is not inherently better.

## Clarity over cleverness (Principle 14)

"Software should be designed for ease of reading, not ease of writing" (Principle 15's
sibling, Principle 14) and the "code should be obvious" principle both say the same thing a
"prefer clarity and correctness over cleverness" directive says: a reader should understand
the behavior and intent of code without significant mental effort, and that's the actual
test of good design - not how compact or clever the implementation is. This applies
identically whether "clever" means a dense one-liner or an unnecessarily elaborate
type-level encoding.

## Keep the interface small, general-purpose

The doctrine gives three questions for sizing an interface: what's the simplest interface that
covers current needs, how many places would a more general version actually be used, and is
the general version still easy to use for the common case. "What is the smallest API that
solves the real problem" and "prefer extension points over new end-user methods" are the
same three questions asked in different words - keep the framework/library surface small and
deep rather than shallow and sprawling (the deep-vs-shallow test still applies:
does this new method's interface add real value, or just clutter?).
