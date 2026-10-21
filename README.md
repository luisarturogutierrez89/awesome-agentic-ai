# AI Engineering Levels — a level-2 workflow for Claude Code

A Claude Code marketplace with one plugin, **crew**, packaging a team of
subagents that work under *supervised autonomy* (level 2): the agents plan,
implement, test and review; you approve the plan and validate the result.

Works on server/web codebases and on native iOS and Android projects.

## What the plugin ships

- **8 subagents**: `product-analyst`, `architect`, `coder`, `tester`,
  `reviewer`, `debugger`, `docs-writer`, `security-auditor`.
- **1 orchestration skill** (`level2-workflow`): the plan → validation
  discipline, which Claude loads only when the task is to plan or build.
- **1 entry command** (`/crew:kickoff`): fires the full pipeline for a feature.

See [the plugin's README](plugins/crew/README.md) for measured results from real
issues, the pipeline diagram, and per-platform behavior.

## Install

Two equivalent ways. **From the terminal:**

```bash
claude plugin marketplace add luisarturogutierrez89/awesome-agentic-ai
claude plugin install crew@ai-eng-levels
```

Or **inside a Claude Code session** (start `claude` first, then type):

```
/plugin marketplace add luisarturogutierrez89/awesome-agentic-ai
/plugin install crew@ai-eng-levels
```

Note that `/plugin` only exists INSIDE the session. Typed into your shell it
gives you `zsh: no such file or directory: /plugin` — use the `claude plugin`
variant there.

Restart the session so it loads. To update after a change in the repo:

```bash
claude plugin update crew@ai-eng-levels
```

Use the full ID including the marketplace — a bare `claude plugin update crew`
fails with "Plugin not found". If you installed with `--scope project`, pass the
same flag when updating. And restart the session afterwards: updating the files
on disk does not reload a session that is already running.

## Use it

Start a session in plan mode inside your project and fire the flow:

```bash
claude --permission-mode plan
```

```
/crew:kickoff classify the receipt type before parsing it
```

**Use plan mode.** It is what makes gate 1 real: without it, "don't touch code
until I approve" is only an instruction; with it, edits are blocked by the
harness until you approve. Plan mode is a CLI flag, not an IDE feature, so it
works from any terminal — including alongside Xcode or Android Studio.

You'll see `product-analyst` clarify requirements, then a technical plan you
approve (gate 1). Note who writes it: **in plan mode the plan comes from Claude
Code's native planner, not from `architect`.** In that case `architect` runs
afterwards, reviewing that plan and adding what it lacks — above all the security
constraints that generic planners leave out. Run without plan mode and
`architect` produces the whole plan itself.

Then coder/tester/reviewer execute, with bounded correction loops, so you
validate the diff at the end (gate 2).

### Giving it a design to work from

Images pasted into the chat reach the orchestrator, not the subagents — each one
is a separate context. To let `architect` and `coder` see a mockup, commit it as
a file and point at the path:

```
/crew:kickoff implement issue #52; the mockup is at docs/design/issue-52.png
```

Every agent can open images with `Read`, so they work from the design itself
rather than from a second-hand description of it.

## If you fork it

1. In `.claude-plugin/marketplace.json`: change `owner.name`, `owner.url` and the
   marketplace `name` (`ai-eng-levels`). The `name` must be kebab-case and unique
   to you — each user can register only one marketplace per name.
2. In `plugins/crew/.claude-plugin/plugin.json`: set your `author.name`.
3. Validate before sharing:
   ```bash
   claude plugin validate .
   ```

## Versioning

`plugin.json` carries a `version` field. Bump it on each release so users receive
the update. If you'd rather have them always follow the latest commit, drop the
field and Claude Code will use the commit SHA instead.

## Using it at work (managed teams)

On a machine administered by your company, managed settings may restrict which
marketplaces can be added (`strictKnownMarketplaces`). If it's blocked you won't
be able to add a personal marketplace there: ask whoever administers Claude Code
to allow it, or to distribute it to the whole team via Organization settings >
Plugins. To share it with your team per repo, declare it in the project's
`.claude/settings.json` with `extraKnownMarketplaces` and `enabledPlugins` so it
registers itself once the folder is trusted.

## Notes

- **Keep nothing confidential in this repo** if it's going to be public: no
  internal documents, client specs, credentials or company data. The agents here
  are deliberately generic.
- Treat any third-party plugin as code you run with trust: review it and pin it
  to a version.

## License

MIT.
