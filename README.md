# ClaudeWD40

**Claude Web Development 40**: a can of [Claude Code](https://claude.com/claude-code)
skills for everyday web development. Spray on, loosen stuck workflows.

## Skills

| Skill | What it does |
|-------|--------------|
| [`change-plan-runner`](skills/change-plan-runner/SKILL.md) | Hand Claude a list of changes and walk away. It asks every question up front (each option comes with an example of what happens next), sets up and proves a test environment (test harness, dev server, a connected Chrome, logins) while you're still there, then for each change does **change → test on localhost in Chrome → fix → retest → one local commit**. It ends with a short summary and a full Markdown report, including anything only you can verify after a deploy. Nothing is pushed or deployed. |

## Install

Skills are folders with a `SKILL.md`. Put them in `~/.claude/skills/` for every
project, or in a repo's `.claude/skills/` for that project only.

```bash
git clone https://github.com/alejandruxxug/ClaudeWD40.git
cd ClaudeWD40

# Every project on this machine (symlink, so `git pull` updates it):
ln -s "$PWD/skills/change-plan-runner" ~/.claude/skills/change-plan-runner

# Or a single project:
cp -R skills/change-plan-runner /path/to/project/.claude/skills/
```

Start a new Claude Code session and the skill shows up. It triggers on its own when
you send a plan ("here's the plan, go through these changes and test each one"), or
you can call it by name.

## Requirements

- [Claude Code](https://claude.com/claude-code)
- For browser testing: [Claude in Chrome](https://claude.com/chrome) (other browser
  automation works as a fallback). Without a browser, changes are verified with the
  project's test harness only, and the report says so.
- A git repo with a dev server that runs locally.

## Contributing

Each skill lives in `skills/<name>/` with a `SKILL.md` (YAML frontmatter with `name`
and `description`, then instructions) and optional `references/`, `scripts/` and
`assets/`. Keep skills project-agnostic: they should learn a repo's specifics from its
`CLAUDE.md` and scripts rather than hard-code them.

## License

[MIT](LICENSE)
