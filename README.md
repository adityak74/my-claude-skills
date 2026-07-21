# my-claude-skills

A collection of custom [Claude Code](https://claude.com/claude-code) skills.

Each skill lives in its own directory containing a `SKILL.md` file with YAML
frontmatter (`name`, `description`) followed by instructions Claude follows
when the skill is invoked.

## Installation

Copy (or symlink) a skill's directory into your Claude Code skills folder:

```bash
cp -r <skill-name> ~/.claude/skills/<skill-name>
```

Or clone this whole repo into your skills directory:

```bash
git clone https://github.com/adityak74/my-claude-skills.git ~/.claude/skills/my-claude-skills
```

Invoke a skill with `/<skill-name>` in Claude Code, or let Claude pick it up
automatically when its description matches the task at hand.

## Skills

### [multi-agent-delegator](./multi-agent-delegator/SKILL.md)

Delegate bounded coding work from Claude Code to interactive Codex and
Antigravity sessions running in visible [cmux](https://github.com) terminal
panes, while the main Claude Code session retains architecture decisions,
coordination, review, and integration.

Use it when a coding goal can be split into parallelizable, independently
verifiable chunks (implementation, test authoring, exploration,
documentation, or independent review) that benefit from running
concurrently in isolated git worktrees, with Claude acting as lead
engineer overseeing and merging the results.

**Trigger:** `/multi-agent-delegator <goal>`

## Contributing

New skills should follow the same layout: a directory named after the
skill containing a `SKILL.md` with `name` and `description` frontmatter.
