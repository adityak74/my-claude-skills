# my-claude-skills

A collection of custom [Claude Code](https://claude.com/claude-code) skills,
distributed as a Claude Code plugin marketplace.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

## Installation

Add this repo as a plugin marketplace, then install the plugin:

```bash
claude plugin marketplace add adityak74/my-claude-skills
claude plugin install multi-agent-delegator
claude plugin install graph-blueprint
claude plugin install mac-dev-workstation
claude plugin install project-primer
```

Or, inside a Claude Code session, use the slash commands:

```text
/plugin marketplace add adityak74/my-claude-skills
/plugin install multi-agent-delegator
/plugin install graph-blueprint
/plugin install mac-dev-workstation
/plugin install project-primer
```

Once installed, invoke a skill's trigger command (e.g. `/multi-agent-delegator`)
and Claude will follow its instructions, or let Claude pick it up automatically
when its description matches the task at hand.

## Skills

### [multi-agent-delegator](./skills/multi-agent-delegator/SKILL.md)

Delegate bounded coding work from Claude Code to interactive Codex and
Antigravity sessions running in visible [cmux](https://github.com/manaflow-ai/cmux)
terminal panes, while the main Claude Code session retains architecture
decisions, coordination, review, and integration.

Use it when a coding goal can be split into parallelizable, independently
verifiable chunks (implementation, test authoring, exploration,
documentation, or independent review) that benefit from running
concurrently in isolated git worktrees, with Claude acting as lead
engineer overseeing and merging the results.

**Trigger:** `/multi-agent-delegator <goal>`

### [graph-blueprint](./plugins/graph-blueprint/skills/graph-blueprint/SKILL.md)

Run a broad task as a graph — nodes do the work, edges carry the results:
define the goal, split it into independent pieces, fan out parallel workers on
different angles (research, compare, check, find gaps), verify every finding in
a fresh context that never saw the worker's reasoning, merge what survives, and
synthesize one clear, actionable report.

Use it when a single pass would miss things: research, audits, code or design
reviews, comparisons, migrations, and any "be thorough" request where
unverified worker output would otherwise leak straight into the answer.

**Trigger:** `/graph-blueprint <goal>`

### [mac-dev-workstation](./plugins/mac-dev-workstation/skills/mac-dev-workstation/SKILL.md)

Bootstrap a fresh macOS development workstation with Homebrew, shell PATHs,
nvm/Node, Python, Rust, Ollama, OrbStack, Slack, and the common CLI tools used
in this repo's setup.

Use it when a Mac has just been formatted, the dev environment needs to be
rebuilt from scratch, or a standard workstation stack needs to be restored.

**Trigger:** `/mac-dev-workstation`

### [project-primer](./plugins/project-primer/skills/project-primer/SKILL.md)

Prime a new repository to open source standards in one pass: a top-1% README
with a star-history chart, an MIT license, contributing / code of conduct /
security guidelines, a pull request template, and the GitHub repo description,
homepage, topics, and labels configured through `gh api`.

Use it when a repo has just been created or is about to go public, and the
same day-one setup keeps getting done by hand — empty description, missing
license, default labels, no PR template, a project website nobody links to.

**Trigger:** `/project-primer`

## Repository layout

```text
.claude-plugin/
  marketplace.json    # marketplace manifest listing all plugins in this repo
  plugin.json         # manifest for the multi-agent-delegator plugin (root plugin)
skills/
  multi-agent-delegator/
    SKILL.md          # skill definition (frontmatter + instructions)
plugins/
  graph-blueprint/
    .claude-plugin/
      plugin.json     # manifest for the graph-blueprint plugin
    skills/
      graph-blueprint/
        SKILL.md
  mac-dev-workstation/
    .claude-plugin/
      plugin.json     # manifest for the mac-dev-workstation plugin
    skills/
      mac-dev-workstation/
        SKILL.md
  project-primer/
    .claude-plugin/
      plugin.json     # manifest for the project-primer plugin
    skills/
      project-primer/
        SKILL.md
```

The `multi-agent-delegator` plugin is sourced from the repo root for backwards
compatibility; every additional plugin lives in its own `plugins/<name>/`
directory so it can be installed independently.

## Contributing

New skills should live under `plugins/<plugin-name>/skills/<skill-name>/SKILL.md`,
with `name` and `description` frontmatter, alongside a
`plugins/<plugin-name>/.claude-plugin/plugin.json` manifest. Register any new
plugin in `.claude-plugin/marketplace.json`.

## License

[MIT](LICENSE)
