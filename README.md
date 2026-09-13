# software-design-depth-doctrine-skill
A personal design-review and design-authoring doctrine for fighting software complexity

## Install as a Claude code plugin

```
/plugin marketplace add vim89/software-design-depth-doctrine-skill
/plugin install software-design-depth-doctrine@software-design-depth-doctrine-skill
```

## Install for OpenAI Codex

```bash
codex plugin marketplace add vim89/software-design-depth-doctrine-skill
codex plugin add software-design-depth-doctrine@software-design-depth-doctrine-skill
```

This reuses the same `.claude-plugin/marketplace.json` and `plugin.json` manifests as the Claude
plugin above - no separate packaging needed. Verify with `codex plugin list --marketplace
software-design-depth-doctrine-skill`.

If your Codex build doesn't have the `plugin` subcommand, fall back to copying `AGENTS.md`:

```bash
cp AGENTS.md /path/to/your/project/AGENTS.md
```

See [`.codex/INSTALL.md`](.codex/INSTALL.md) for the submodule option and usage notes.
