# Software design depth doctrine for OpenAI Codex

## Installation

### Option 1: Copy AGENTS.md

```bash
# From the software-design-depth-doctrine-skill repository
cp AGENTS.md /path/to/your/project/AGENTS.md
```

This gives Codex the full doctrine, the 15 principles, and the red-flags checklist inline. The
reference files it points to (deep modules, decomposition, comments/naming, trends, typed-FP
translations) stay in this repository - keep the repo cloned alongside the copied `AGENTS.md` if you
want Codex able to read them, or copy the `plugins/software-design-depth-doctrine/skills/software-design-depth-doctrine/references/`
directory too.

### Option 2: Reference as submodule

```bash
cd your-project
git submodule add https://github.com/vim89/software-design-depth-doctrine-skill.git .design-doctrine
```

Then reference it from your project's own `AGENTS.md`:

```markdown
# Project agents

See `.design-doctrine/AGENTS.md` for the design-review and design-authoring doctrine
(deep modules, information hiding, error handling, naming, comments) to apply to every diff -
human- or AI-generated.
```

## What's included

- The complexity model (`C = Σ (cp · tp)`, two root causes: dependencies and obscurity)
- The 15 design principles, compressed to what should change in your next review comment
- A 14-row red-flags checklist to run against any diff in about five minutes
- Reference files with full detail and code examples, organized by theme

## Usage

After installation, ask Codex things like:

- "Is this module deep or shallow?"
- "Review this diff against the design-depth doctrine."
- "Where is this class leaking implementation details into its interface?"
- "Does this comment describe something the code doesn't already say?"

Codex will apply the doctrine's red flags and principles directly from `AGENTS.md`, and pull in a
reference file from `plugins/software-design-depth-doctrine/skills/software-design-depth-doctrine/references/`
when the task needs more depth than the quick reference covers.
