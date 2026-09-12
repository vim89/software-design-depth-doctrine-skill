# Decomposition and errors

## Table of contents
- Better together or better apart: the merge/split decision
- Repetition (red flag)
- Special-general mixture (red flag)
- Conjoined methods (red flag)
- Worked examples: NetworkErrorLogger, History/undo
- Splitting and joining guidance
- Define errors out of existence
- Exception cost / masking / aggregation
- Design it twice

## Better together or better apart

There's no universal rule for whether two pieces of functionality should live in one module or be split across two - the decision should be driven by which arrangement produces simpler interfaces and less duplicated knowledge, not by superficial size targets.

**Signals favoring merging two things together:**
- They share information (merging avoids re-exposing that shared information across an interface boundary) - e.g., merging HTTP-request *reading* and *parsing* into one class, since reading a request already requires partially parsing it (see the information-leakage case study in `deep-modules-and-information-hiding.md`).
- They are always used together, and no one uses one without the other.
- There's significant overlap in what they conceptually do (e.g., simplifying the combined interface by removing duplicate concepts).

**Signals favoring splitting things apart:**
- Separating general-purpose mechanism from special-purpose policy (see "Special-General Mixture" below).
- The pieces are logically independent and combining them would only add unrelated concerns to a single class's interface.

**Red flag: Repetition** - the same nontrivial piece of code shows up over and over. This is a strong signal that some knowledge or mechanism has not been factored into its own module - repetition and information leakage are closely related symptoms of the same underlying problem (a piece of knowledge not living in exactly one place).

**Red flag: Special-General mixture** - special-purpose code (logic serving one particular caller or scenario) is tangled into code that is meant to be general-purpose, contaminating the general mechanism with assumptions that don't belong there. Worked example: a text editor's cursor/selection logic - general text-storage mechanisms should not embed assumptions specific to how a UI displays a cursor or selection; that logic belongs in a separate, special-purpose layer built on top of the general one.

**Anti-example - `NetworkErrorLogger`**: a class that mixed a general-purpose logging mechanism with special-purpose knowledge of network error formats, resulting in a class that was neither a clean general logger nor a clean network-specific handler - illustrating how mixing these concerns produces a class that's awkward for both use cases.

**Worked example - `History` class and undo**: refactoring an undo mechanism revealed that generic "record and reverse an action" logic (general-purpose) had become entangled with knowledge of *which specific actions* existed in the application (special-purpose) - separating these cleanly (a general `History` mechanism, with special-purpose action objects plugged into it) is a canonical example of the general/special separation this chapter argues for.

**Red flag: Conjoined methods** - two methods have so many hidden dependencies on each other that you cannot understand the implementation of one without also understanding the implementation of the other, even though nothing about their signatures signals that coupling. This differs from simple repetition - the issue isn't duplicated code but an invisible, undocumented coupling between two supposedly independent pieces.

**Guidance on splitting vs. joining methods**: when a method has grown to do several logically distinct things, consider whether splitting it produces sub-methods with genuinely independent, understandable interfaces - if splitting only produces two halves that still can't be understood without each other (conjoined methods again), splitting hasn't actually reduced complexity, just relocated it.

## Define errors out of existence

Exceptions and special-case error handling are one of the largest, most underestimated sources of complexity in real systems - the "happy path" logic is often simple, but the number of distinct ways things can fail, and the code needed to handle each one, tends to dominate a system's actual complexity.

Guiding question when designing error handling: **can this error condition be designed out of existence entirely**, rather than caught and handled? Concrete illustrations:

- **Tcl `unset` mistake**: an early Tcl design threw an error when unsetting a variable that didn't exist - in hindsight, unnecessary; unsetting something already gone could simply be a no-op, eliminating an entire class of caller-side error handling for no loss of correctness.
- **File deletion - Unix vs. Windows**: Unix allows deleting a file that's still open elsewhere (the file's storage is only reclaimed once the last reference closes); Windows classically disallowed this, forcing every caller to handle a "file in use" error. Unix's approach designs the error condition out of existence by making the operation itself well-defined without needing that restriction.
- **`substring` critique**: since Scala's `String` is backed by `java.lang.String`, throwing exceptions for out-of-range indices in `substring` is criticized as a case where clamping to valid bounds automatically (rather than raising an exception) would have been a perfectly sensible, error-eliminating default for the common case - contrast with Scala's own collection methods like `slice`, `take`, and `drop`, which already clamp out-of-range indices instead of throwing.
- **Masking exceptions** - sometimes the cleanest way to "eliminate" an error at a given layer is to catch and handle it at a *lower* layer so it never propagates up as something callers must think about. Examples: TCP masks lost/corrupted packets from applications by silently retransmitting; NFS-style systems can mask certain network hiccups from the application layer entirely.
- **Exception aggregation** - instead of defining a distinct, finely-typed exception for every possible failure (and forcing callers to handle each distinctly), group many different failure causes under one broad, coarser exception the caller handles uniformly. RAMCloud's crash-recovery design is cited as an example: rather than distinguishing every possible individual failure mode, the system treats many of them uniformly as "this operation must be retried/recovered," collapsing what would otherwise be many special cases into one handled path.
- **"Just crash" as a legitimate design choice** - for truly unrecoverable, rare conditions, deliberately crashing (rather than building elaborate handling for a case that can't be meaningfully recovered from anyway) can be the simplest, most honest option - assuming the failure is genuinely rare and recovery isn't actually possible.
- **Designing special cases out of existence (not just errors)** - the same philosophy applies beyond exceptions: e.g., a text-selection implementation that treats "no selection" as a zero-length selection (rather than a distinct null/absent state) can eliminate an entire branch of special-case logic throughout the codebase, because now every code path can assume a selection object always exists.

**Taking it too far**: masking or aggregating errors can be overdone. A cautionary example: a student team masked *all* network exceptions indiscriminately (treating every network failure as recoverable/ignorable), which hid real, actionable failures the application actually needed to respond to differently. The judgment call is whether an error is genuinely irrelevant to the caller, not whether hiding it is convenient.

## Design it twice

Before settling on a design, sketch at least two substantially different alternatives and compare their tradeoffs - even if the first idea seems obviously fine. This applies at multiple levels: interface design, implementation approach, UI design, and even system-level decomposition into modules.

Worked illustration: comparing multiple candidate interfaces for a text-editing class surfaces tradeoffs (e.g., simplicity for common operations vs. flexibility for less common ones) that aren't visible when only one design is ever considered - having a second option to compare against is often what reveals a first design's hidden weaknesses.

The extra time spent designing twice is justified because early design mistakes are the most expensive to fix later - the cost of exploring a second option up front is small relative to the cost of discovering a bad interface after callers depend on it.

Notably, this technique meets resistance even from experienced, capable engineers - there's a natural pull toward finalizing the first workable idea rather than deliberately generating a second, different one to compare it against; this resistance is itself worth recognizing and pushing past.
