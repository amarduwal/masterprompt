# masterprompt

Two Claude Code skills that start any web or mobile project properly, new or existing.

| Skill | What it does |
|---|---|
| `project-docs` | Interviews you in three gated rounds (idea, scope, tech), then writes the full docs set in `docs/`: BRIEF, PRD, SRS, UX, ARCHITECTURE, TECH_SPEC, SECURITY, TEST_PLAN, PLAN, DECISIONS, and one spec per feature. |
| `project-bootstrap` | Reads those docs and sets up persistent context so no session or teammate has to re-explain the project: `CLAUDE.md`, path-scoped rules, project skills (`/next-task`, `/handoff`, `/rules-check`, and more), shared settings and hooks, a reviewer agent, and infrastructure sized to the project's tier, assessed against 111 production concerns. |

Both stop at confirmation gates and write nothing before you approve. Neither commits.

## Install

In Claude Code:

```
/plugin marketplace add amarduwal/masterprompt
/plugin install project-kit@masterprompt
```

Or from a shell (`--scope user` makes it available in every project):

```
claude plugin marketplace add amarduwal/masterprompt
claude plugin install project-kit@masterprompt --scope user
```

## Use

From the project's root folder:

```
/project-kit:project-docs I'm building an app that ...
/project-kit:project-bootstrap
```

Plugin skills are namespaced, so keep the `project-kit:` prefix. Text after the command is optional context.

When bootstrap finishes, the project has its own skills. Use `/next-task` to build the next task in `docs/PLAN.md` and `/handoff` at the end of a session.

Without installing anything, you can paste the body of either `SKILL.md` into Claude Code as a prompt.

## Update

Auto-update is off for third-party marketplaces. To get the latest version:

```
/plugin marketplace update masterprompt
```

Or turn on auto-update in `/plugin`, under the Marketplaces tab.

## Layout

```
.claude-plugin/marketplace.json           marketplace catalog
plugins/project-kit/
  .claude-plugin/plugin.json              plugin manifest
  skills/project-docs/SKILL.md
  skills/project-bootstrap/SKILL.md
```

## Maintaining

- Edit `plugins/project-kit/skills/*/SKILL.md`. On the author's machine, `~/.claude/skills/project-docs` and `~/.claude/skills/project-bootstrap` are symlinks to these folders, so edits apply immediately as `/project-docs` and `/project-bootstrap`. Don't also install the plugin on that machine, or each skill shows up twice.
- Validate before pushing:

  ```
  claude plugin validate . && claude plugin validate ./plugins/project-kit
  ```

  The "No version specified" warning is expected (see the next point).

- No `version` is set, so every pushed commit is a new version for anyone who updates. To ship pinned releases instead, add `"version": "1.0.0"` to `plugin.json` only (not `marketplace.json`) and bump it on each release.
- Private repo: teammates need non-interactive git access to it (`gh auth login` then `gh auth setup-git`, or an SSH key loaded in ssh-agent).

## License

MIT, see [LICENSE](LICENSE).
