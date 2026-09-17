# that-mathevs-skills

Agent skills for [Claude Code](https://claude.com/claude-code). Opinionated, small, and meant to be used — each one earns its place by changing how the agent works, not by describing what it already does.

## Install

```sh
claude plugin marketplace add that-mathevs/that-mathevs-skills
claude plugin install that-mathevs-skills@that-mathevs
```

Restart Claude Code and the skills are available. Check what landed:

```sh
claude plugin details that-mathevs-skills
```

To update later, refresh the marketplace, then update the plugin, and restart Claude Code:

```sh
claude plugin marketplace update that-mathevs
claude plugin update that-mathevs-skills
```

The first command only fetches the new listing. The second installs the new version.

To remove, `claude plugin uninstall that-mathevs-skills`.

### One skill only

Every skill is a self-contained folder, so you can take just one without the plugin:

```sh
git clone https://github.com/that-mathevs/that-mathevs-skills.git
cp -R that-mathevs-skills/skills/bdd ~/.claude/skills/
```

Anything in `~/.claude/skills/` loads on the next session. Copy it into a project's `.claude/skills/` instead to scope it to that repo.

## The skills

### `bdd` — behaviour-driven development

Tests that read as a specification. Covers the scenario as the unit of work (given / when / then), naming in the project's own domain language, outside-in development, and the five ways a test name goes wrong.

Its completion criterion is the point of it:

> Print the test names and read them as a document. A stranger should learn what the system does from that list alone, without opening the code.

Pairs with the `tdd` skill from [mattpocock/skills](https://github.com/mattpocock/skills), which owns the red → green loop. `bdd` owns what each test *says*, and points at `tdd` for the mechanics rather than restating them.

| File | What it holds |
|---|---|
| `SKILL.md` | The core: the scenario, ubiquitous language, outside-in, anti-patterns |
| `naming.md` | The naming grammar, with before/after tables |
| `scenarios.md` | Worked scenarios in `node:test`, Vitest and Gherkin; example tables; fakes |

## Layout

```
.claude-plugin/
  marketplace.json   # makes this repo an installable marketplace
  plugin.json        # the plugin, and the skills it ships
skills/
  <name>/
    SKILL.md         # frontmatter (name, description) + the skill itself
    *.md             # reference files, reached by pointers from SKILL.md
```

The repo is both the marketplace and the plugin, so `marketplace add` and `plugin install` both point at the same place.

## Adding a skill

1. `mkdir skills/<name>` and write `SKILL.md`, starting with frontmatter:

   ```markdown
   ---
   name: <name>
   description: <what it is>. Use when <trigger>, when <trigger>, or when <trigger>.
   ---
   ```

   The `description` is the skill's context pointer: it sits in the agent's context permanently and decides when the skill fires, so the triggers in it matter more than the prose. List genuinely distinct cases, not synonyms for one case.

2. Push reference material into sibling `.md` files and point at them from `SKILL.md`, so the main file stays legible and the extra pages load only when they're reached.

3. Add the path to `skills` in `.claude-plugin/plugin.json`.

4. Test it before committing, by symlinking into the live skills directory:

   ```sh
   ln -s "$PWD/skills/<name>" ~/.claude/skills/<name>
   ```

   Edits then apply on the next session with no reinstall. Remove the symlink before installing the published plugin, or the same skill loads twice.

Skills that only ever fire when you type their name can set `disable-model-invocation: true`, which drops the description from the agent's context and costs nothing until invoked.

## Licence

MIT — see [LICENSE](LICENSE).
