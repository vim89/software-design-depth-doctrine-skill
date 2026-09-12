# Software trends and performance

## Table of contents
- The evaluation yardstick for any new methodology
- Object-oriented programming and inheritance
- Agile development
- Unit tests
- Test-driven development
- Design patterns
- Getters and setters
- Designing for performance
- Conclusion: the reward for good design

## The evaluation yardstick

Whenever a new methodology, pattern, or process is proposed, evaluate it against one question: **does this actually reduce dependencies or obscurity in large systems, or does it just look rigorous on the surface?** Many popular practices sound self-evidently good but, examined against this yardstick, can make complexity worse rather than better. What follows applies this test to several well-known trends, then to performance work specifically.

## Object-oriented programming and inheritance

OOP mechanisms (classes, private members, inheritance) can support good design, but don't guarantee it - a class built with OOP features can still be shallow or leak information.

- **Interface inheritance** (a parent defines method signatures only; each subclass implements them independently) is a genuine complexity win: it lets one interface serve multiple implementations (e.g., an I/O interface implemented for both disk files and network sockets), and the more implementations an interface supports, the deeper it effectively becomes.
- **Implementation inheritance** (subclasses inherit a parent's default method bodies, optionally overriding them) reduces code duplication but creates a real cost: it creates dependencies between parent and every subclass. A change to the parent may require re-examining every subclass; a subclass override may require understanding the parent's implementation. In the worst case, understanding any one class in a hierarchy requires understanding the whole hierarchy. **Use implementation inheritance cautiously** - consider composition (small helper classes providing shared functionality) as an alternative first. If implementation inheritance is unavoidable, try to cleanly separate which state belongs to the parent vs. the subclasses (e.g., subclasses treat parent-managed instance variables as read-only), applying information hiding *within* the class hierarchy.

## Agile development

Agile is mostly a process methodology (team structure, scheduling, customer interaction), not a design methodology, but it touches design indirectly. Its incremental, iterative philosophy aligns with this book's view that a good design usually can't be fully visualized upfront and should emerge across iterations.

**Risk**: agile can slide into pure tactical programming - its emphasis on "just enough to ship the next feature" can discourage investing in general-purpose abstractions upfront, encouraging developers to defer design decisions indefinitely rather than making them deliberately when the need first becomes clear.

Key reframing: **the increments of development should be abstractions, not features** (Principle 15). It's fine to defer designing an abstraction until it's actually needed - but once it's needed, invest real design effort in it (including making it somewhat general-purpose), rather than bolting on the minimum needed for just the current feature.

## Unit tests

Unit tests (small, isolated, per-method) differ from system/integration tests (whole-application, usually owned by a separate QA function). Their design-relevant value is that **they make refactoring safe** - without a strong test suite, developers rationally avoid structural changes because there's no fast way to detect regressions, so complexity accumulates unchecked and design mistakes never get corrected. A strong test suite converts refactoring from a risky bet into a routine, safe activity.

Illustrative example: rewriting Tcl's interpreter into a byte-code compiler was a sweeping, invasive change across the entire core engine - the existing unit test suite caught nearly all resulting bugs before release, with only a single bug surfacing post-release.

## Test-driven development

The author explicitly is **not** a fan of TDD, despite being a strong advocate of unit testing generally. The critique: TDD focuses attention on making the next test pass, not on finding the best overall design - it is inherently tactical and overly incremental, with no natural checkpoint at which to step back and design. This mirrors the general critique of feature-by-feature increments in agile: TDD applies that same failure mode at an even finer grain.

**One legitimate exception**: when fixing a bug, write a failing test that reproduces the bug *first*, then fix the bug and confirm the test passes. This is the only reliable way to confirm the fix actually addresses the reported bug (fixing first risks writing a test that doesn't actually exercise the failure).

## Design patterns

Design patterns (from *Design Patterns: Elements of Reusable Object-Oriented Software*) are broadly beneficial because they encode well-understood, previously-validated solutions to recurring problems. **The main risk is over-application**: forcing a problem into a familiar pattern when a custom, purpose-fit approach would actually be cleaner. More design patterns applied is not inherently better - the same logic as "more classes isn't inherently better" (see classitis) applies here.

## Getters and setters

Getter/setter methods (`getFoo`/`setFoo`) are a Java convention, born from Java's lack of a uniform access principle. Scala doesn't need this boilerplate: a public `val`/`var` field already looks identical to a method call at the use site, so validation, side effects, or notification logic can be added later (via `def foo: T` / `def foo_=(v: T): Unit`) without changing any caller. When Scala code replicates the Java `getFoo`/`setFoo` pattern anyway, it's usually an unreflective port of Java habits into a language that already solved this problem. Critique: such methods are **shallow methods** almost by definition (typically one line), adding interface clutter without much functionality. The stronger fix is to avoid exposing instance-variable-equivalent state in the first place, rather than wrapping exposed state in a getter/setter pattern. As with design patterns generally, the popularity of the pattern itself has led to overuse when carried over without reconsidering whether it's still needed.

## Designing for performance

**How to think about performance**: certain operation classes are reliably expensive regardless of context - network round-trips, disk I/O, memory allocation, and cache misses chief among them. Concrete points: hash tables vs. ordered maps have different performance profiles depending on access pattern; RAMCloud's use of kernel-bypass networking is cited as an example of eliminating an expensive layer entirely rather than optimizing within it. Key claim tying performance back to the whole book: **simpler code tends to run faster** - clean, deep-module designs with less incidental complexity often outperform "clever" but tangled code, not despite being simple but because of it.

**Measure before modifying**: don't trust intuition about where time is going in a system - get a real baseline and targeted measurements before optimizing, since intuitive guesses about hot paths are frequently wrong.

**Design around the critical path**: define what "the ideal," minimal-work version of an operation would look like on its most common path, then design real code that approximates that ideal as closely as possible using clean abstractions - rather than optimizing after the fact. A recurring pattern: do special-case checks once, up front, rather than scattering them repeatedly along the hot path.

**Extended example - RAMCloud `Buffer` class**: a real optimization pass that both roughly doubled throughput and reduced code size by about 20%, by redesigning the buffer's critical path around the ideal-case flow (including an `extraAppendBytes` optimization) rather than patching the existing structure - direct evidence that better design and better performance are not in tension.

**Conclusion**: clean design and high performance are compatible goals, not competing ones - a well-designed system is frequently also the faster one, because unnecessary complexity (extra layers, extra checks, extra allocations) is itself often a performance cost.

## Conclusion: the reward for good design

The book's closing argument reframes the investment mindset as a source of professional enjoyment, not just correctness. Root causes (dependencies, obscurity), red flags (leakage, unneeded errors, generic names), and general techniques (deep/generic classes, defining errors out of existence, separating interface from implementation documentation) all serve one end: reducing the drudgery of chasing bugs in brittle, poorly-understood code.

The explicit tradeoff: good design costs more time early (and costs even more if you're still learning the underlying skills), which can feel like it's getting in the way if the only goal is "make it work right now." But the investment pays off quickly - well-designed modules get reused profitably, clear documentation saves real time on every future return to the code, and design skill itself compounds with practice so that good designs stop taking meaningfully longer than quick, tactical ones. The final reward: skilled designers spend a larger fraction of their time in the genuinely enjoyable design phase, while poor designers spend most of their time fighting bugs in complicated, brittle code - making good design not just a quality outcome but a more sustainable and enjoyable way to work.
