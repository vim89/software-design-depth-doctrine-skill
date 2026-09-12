# Deep modules and information hiding

## Table of contents
- Modular design and the interface/implementation split
- Abstraction: what it is and how it can be wrong
- Deep vs. shallow modules
- Classitis
- Example: Scala/Java I/O stacking vs. Unix I/O
- Information hiding
- Information leakage (red flag)
- Temporal decomposition (red flag)
- HTTP server case study
- Overexposure (red flag)
- General-purpose modules are deeper
- Different layer, different abstraction
- Pass-through methods (red flag)
- Pulling complexity downward

## Modular design and the interface/implementation split

Every module (class, function, subsystem, service) has an **interface** (what a developer must know to use it) and an **implementation** (how it fulfills that interface). Developers working on a module must understand its interface and implementation, plus the interfaces of everything it calls - but never the implementations of those other modules. The goal of modular design is to minimize the dependencies between modules by keeping interfaces simple relative to what they accomplish.

Interfaces have a **formal** part (checked by the language: method signatures, types, exceptions) and an **informal** part (behavior, ordering constraints, assumptions) which can only be conveyed through comments. For most interfaces the informal part is larger and more important than the formal part.

## Abstraction

An abstraction is a simplified view of an entity that omits unimportant details. The word "unimportant" is the crux: an abstraction goes wrong in one of two directions - including details that don't actually matter (adds needless cognitive load) or omitting details that do matter (creates obscurity - the abstraction looks simple but is actually incomplete, i.e. a "wrong abstraction"). Designing a good abstraction means figuring out what's truly important and minimizing that set.

## Deep vs. shallow modules

Picture each module as a rectangle: area = functionality provided, top edge width = interface complexity. **Deep modules** have a large area and a narrow top edge - lots of functionality behind a simple interface. **Shallow modules** have an interface that's complex relative to the (small) functionality they provide.

- Classic deep-module example: **Unix file I/O** - five simple calls (`open`, `read`, `write`, `lseek`, `close`) hide hundreds of thousands of lines of implementation complexity (disk layout, permissions, caching, scheduling), and that interface has remained essentially unchanged for decades even as the implementation was rewritten many times.
- Another deep-module example: a **garbage collector** - it has essentially no interface at all (it actually *shrinks* the language's interface by removing the need for manual `free`), yet does enormous, complex work invisibly.
- Extreme shallow-module example: a one-line method like `addNullValueForAttribute(attribute) { data.put(attribute, null); }` - it adds a new name developers must learn without hiding any complexity or reducing the caller's work. If documented properly, the comment for such a method would be longer than the method itself.

**Red flag: Shallow module** - the interface for a class or method isn't much simpler than its implementation. Shallow modules don't help against complexity: the benefit (not having to learn internals) is cancelled out by the cost of learning and using the interface itself. Small modules tend to be shallow.

## Classitis

A cultural bias (reinforced by advice like "classes should be small" or "methods over N lines should be split") pushes developers toward many small, shallow classes - "classitis": the false belief that "classes are good, so more classes are better." Classitis increases *total system complexity* even though each individual class looks simple, because the number of interfaces (each carrying its own cognitive load) multiplies faster than functionality does.

## Example: Scala/Java I/O stacking vs. Unix I/O

Opening a file for buffered, serialized-object I/O by wrapping Java's I/O classes from Scala classically requires stacking three separate objects (`new ObjectInputStream(new BufferedInputStream(new FileInputStream(path)))`), where the first two are discarded after setup and buffering must be explicitly opted into - an easy mistake to omit, silently costing performance. Contrast this with Unix I/O, where buffering (and sequential access) is the automatic default, and the rare need for something different (random access via `lseek`) is available without complicating the common path - or with Scala's own `scala.io.Source.fromFile(path).getLines()`, where buffered, line-oriented reading is the default and no wrapper stacking is needed for the common case. Lesson: **interfaces should be designed to make the common case as simple as possible** - even if that means the interface intentionally isn't "clean" in a factored, orthogonal sense.

## Information hiding

The most important technique for making modules deep. Each module should encapsulate specific pieces of knowledge (design decisions) inside its implementation, invisible from its interface. Examples of information worth hiding: how a B-tree balances itself, how logical file blocks map to physical disk blocks, TCP protocol internals, JSON parsing details.

Information hiding reduces complexity two ways: it simplifies the interface (reduces cognitive load), and it isolates the impact of future design changes to a single module (a TCP implementation change shouldn't ripple into code that merely sends/receives data over TCP).

Important nuance: **making variables/methods `private` is not the same as information hiding.** A private field exposed indirectly through public getters/setters is just as "leaked" in practice as if it were public - the information about its existence and semantics is still visible externally.

Partial information hiding still has value: if a rarely-needed feature is exposed through a separate, rarely-used method (so the majority of callers never encounter it), that's a form of hiding even though the information technically exists in the interface.

## Information leakage (red flag)

**Red flag: Information leakage** - the opposite of information hiding: the same design decision is reflected in more than one module, creating a dependency between them (change one, must change all). If leaked information appears in a module's interface, it's leakage by definition - but leakage can also happen "through the back door," e.g. two classes both hard-code assumptions about the same file format without either exposing that fact in its interface. Back-door leakage is more dangerous because it's not obvious.

The fix, once leakage is spotted: ask "how can I reorganize these classes so this piece of knowledge affects only one class?" Options: merge the classes if they're small and tightly coupled around the leaked knowledge, or extract the shared knowledge into a new class - but only if that new class can expose a genuinely simple interface; otherwise you've just relocated the leakage.

## Temporal decomposition (red flag)

**Red flag: Temporal decomposition** - a common cause of information leakage: structuring code around the *order operations happen in* ("first read, then modify, then write") rather than around the *knowledge* each piece needs. Order is usually a real constraint that must exist somewhere in the system, but it should not dictate the module boundaries unless the different stages truly use different information. Focus module boundaries on "what knowledge does this task need," not "when does this task run."

## HTTP server case study (worked example)

Drawn from a software design course where students implemented HTTP request/response handling:

- **Mistake - too many classes**: one team split "read the request from the socket" and "parse the request" into two classes. This is temporal decomposition - reading actually requires partial parsing (e.g., `Content-Length` must be parsed to know how many body bytes to read), so both classes ended up duplicating knowledge of the HTTP format. Fix: merge into one class. This illustrates a broader theme: **information hiding can sometimes be improved by making a class larger** - either to bring all code for one capability together, or to raise the interface's level of abstraction (one method instead of three sequential ones).
- **Good choice - parameter handling**: student code correctly hid two things from callers: whether a parameter came from the URL or the body (merged into one lookup), and URL-encoding (decoded automatically before being handed back).
- **Bad choice - shallow parameter accessor**: `def params: Map[String, String] = internalMap` returns the internal storage map directly - a shallow method that exposes the internal representation (any future change to that representation breaks all callers) and forces two-step lookups. Better: `def parameter(name: String): Option[String]`, plus typed variants like `def intParameter(name: String): Option[Int]` - deeper, hides the representation, and saves callers a manual conversion step.
- **Mistake - inadequate defaults**: one team required callers to explicitly pass the HTTP version when constructing a response, even though that information is knowable from the request already in hand - forcing the caller to supply information the library could infer, and risking leakage if the caller supplies it wrong. General principle: whenever possible, a class should "do the right thing" automatically, without being asked - the best features are the ones users never have to know exist.

## Overexposure (red flag)

**Red flag: Overexposure** - an API forces callers to learn about rarely-used features just to use the commonly-used ones, raising the cognitive load of the common case. If most developers only need a handful of an interface's features, the interface's *effective* complexity should be scoped to just those features, with rarely-needed capabilities kept out of the way.

## Information hiding within a class

The same principle applies below the class-interface level: design private methods so each hides a specific piece of internal knowledge from the rest of the class, and try to minimize how many places within the class each instance variable is touched.

## Taking information hiding too far

Information hiding only makes sense for information that genuinely isn't needed outside the module. If external callers legitimately need certain configuration to be tunable, that information must be exposed, not hidden for its own sake. The goal is to *minimize* what's needed externally (e.g., prefer a module that can auto-tune its own configuration) - not to hide everything indiscriminately.

## General-purpose modules are deeper

Aim for the "somewhat general-purpose" sweet spot: neither so special-purpose that it only serves today's exact use case, nor so generic that it serves no one well. Example: a GUI text editor's backspace/delete logic - instead of writing narrow methods like `backspace()` and `deleteKeyPressed()` tied directly to specific key bindings, expose a general `deleteRange(start, end)` mechanism, which is both simpler to implement *and* serves more callers (backspace, delete, cut, and future callers) without new interface surface.

Three questions to ask when deciding how general to make an interface:
1. What is the simplest interface that covers all my current needs?
2. In how many places would this be used if it were more general-purpose?
3. Is the resulting interface easy to use for the current use cases, even though it's more general?

## Different layer, different abstraction

Systems are organized into layers (e.g., UI, business logic, storage). Each layer should represent a *different* abstraction than the one above and below it. If a layer's abstraction looks the same as the layer next to it, that's a signal the layering itself may be unnecessary.

**Red flag: Pass-Through method** - a method that does almost nothing except forward its arguments to another method with a nearly identical signature. Pass-through methods indicate the two layers are not actually offering different abstractions - they're duplicating an interface across a layer boundary. Example: a `TextDocument` method that just forwards to an internal storage object's near-identical method without adding meaning.

Exceptions where pass-through-like calls are fine: **dispatchers** (choosing among multiple possible implementations behind one call, e.g. virtual dispatch) and **decorators** (e.g., `scala.io.BufferedSource` wraps a raw `InputStream` and deliberately adds one clear capability - buffering, exposed through the same `Source` interface). Decorators still carry a risk: if overused, they degenerate into classitis, with many thin wrapper layers each adding little.

**Pass-through variables**: a variable that's passed down through many layers of calls just so a lower layer can use it, even though the intermediate layers never touch it - this is a form of information leakage across layers. Consider a **context object** that carries shared, cross-cutting state instead of threading the same parameter through every signature in the chain.

## Pulling complexity downward

When complexity must exist somewhere, prefer pulling it *down* into the implementation of a module rather than pushing it *up* into every caller - because a module has one implementation but potentially many callers, so the payoff of hiding complexity in the implementation is much higher than exposing it in the interface. Example: an editor's text class should absorb complexity around change tracking internally rather than pushing bookkeeping obligations onto every caller.

Critique of **configuration parameters** as a design habit: exposing a knob is often a way of avoiding the harder work of figuring out a sensible default or an adaptive mechanism, effectively pushing complexity the module's author didn't want to solve onto every single caller.

**Taking it too far**: pulling complexity downward can be carried to an extreme where one class absorbs everything and becomes an unmanageable god-class. The principle is a bias, not an absolute - it must be balanced against keeping individual modules deep but still coherent.
