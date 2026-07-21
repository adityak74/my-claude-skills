# my-claude-skills

A collection of custom [Claude Code](https://claude.com/claude-code) skills,
distributed as a Claude Code plugin marketplace.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

## Installation

Add this repo as a plugin marketplace, then install the plugin:

```bash
claude plugin marketplace add adityak74/my-claude-skills
claude plugin install multi-agent-delegator
```

Or, inside a Claude Code session, use the slash commands:

```text
/plugin marketplace add adityak74/my-claude-skills
/plugin install multi-agent-delegator
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

## Repository layout

```text
.claude-plugin/
  marketplace.json   # marketplace manifest listing all plugins in this repo
  plugin.json         # manifest for the multi-agent-delegator plugin
skills/
  multi-agent-delegator/
    SKILL.md          # skill definition (frontmatter + instructions)
```

## Contributing

New skills should live under `skills/<skill-name>/SKILL.md`, with `name` and
`description` frontmatter. Register any new plugin in
`.claude-plugin/marketplace.json`.

## License

[MIT](LICENSE)
