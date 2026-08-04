# vscode-agents

Custom VS Code Copilot agents (`.agent.md`), adapted from
[felipebalbi/opencode-setup](https://github.com/felipebalbi/opencode-setup).

A baseline team of nine specialists, kept in one git repo and loaded into
**every** workspace rather than copied per-project.

## The agents

| Agent | Edits code? | Role |
| --- | --- | --- |
| `architect` | no | API and module-boundary design; produces specs, not code |
| `blog` | yes | Narrative first-person write-ups; `docs` with a voice |
| `coder` | yes | Translates specifications into focused, scoped patches |
| `coordinator` | no | Decomposes work, routes to specialists, tracks deps |
| `docs` | yes | README, rustdoc, tutorials, architecture explainers (Diataxis) |
| `integrator` | yes | Stages, commits, drafts PRs; never force-pushes |
| `reliability` | no | Failure-mode analysis; recovery and observability gaps |
| `reviewer` | no | Adversarial correctness review; evidence-driven verdict |
| `tester` | yes | Edge-case discovery, repros, regression tests |

All nine are usable both from the agent picker and as subagents. `coordinator`
is the only one with the `agent` tool, and it can dispatch the other seven.

Handoff buttons wire the common flows: architect → coder, coder → reviewer /
tester / integrator, reviewer → coder / architect, tester → coder,
reliability → architect / coder, docs → reviewer, blog → reviewer / coder.

`blog` and `docs` deliberately disagree. `docs` enforces Diataxis purity and
reference-grade neutrality; `blog` braids modes, writes in first person, and
treats the dead end as the content. Both keep the same hard rules: code must
compile, no inventing APIs, mandatory drift check.

## Install

Clone anywhere, then symlink the agent files into `~/.copilot/agents/`, which
VS Code scans by default — no settings required:

```bash
mkdir -p ~/.copilot/agents
ln -sfn "$PWD/agents/"*.agent.md ~/.copilot/agents/
```

Reload the window. The agents appear in the chat agent picker and become
available as subagents in every workspace. Because these are symlinks, `git
pull` propagates edits instantly; re-run the loop only when a *new* agent file is
added.

This location is also read by Copilot CLI and background agents, so one clone
serves all three.

### Alternative: `chat.agentFilesLocations`

If you prefer a settings-driven install, point VS Code at the `agents/`
directory:

```jsonc
{
    "chat.agentFilesLocations": {
        "/absolute/path/to/vscode-agents/agents": true
    }
}
```

**On remote workspaces this must go in your local User settings or in Remote
settings** (`Preferences: Open Remote Settings`). It does *not* work in
`~/.vscode-server/data/User/settings.json` — that directory holds extension-host
state only and is never read as configuration. The path itself resolves on the
remote host, so the clone must exist there.

## Per-project overrides

A workspace file at `.github/agents/<name>.agent.md` takes precedence over the
global file of the same name. There is no field-level merge — the workspace file
is the entire agent definition for that project.

Seed an override from the baseline, add the project-specific bits (trigger
keywords in `description`, hard rules, citations of `AGENTS.md` /
`CONTRIBUTING.md` / CI scripts), and commit it in the project repo. Delete the
override when it drifts back toward the baseline; the global file takes over
automatically.

## Notes

- Tool restrictions replace opencode's `permission:` blocks. Read-only agents
  (`architect`, `coordinator`, `reliability`, `reviewer`) simply have no `edit`
  tool.
- opencode's per-command bash allow/ask lists have no direct frontmatter
  equivalent. `integrator` encodes its git policy as hard rules in the body. To
  enforce it deterministically, use `chat.tools.terminal.autoApprove` or a
  `PreToolUse` hook.
- `~/.copilot/` is per-machine, so repeat the install on each host you work
  from. The same directory also scans `instructions/`, `skills/`, and `hooks/`.

## Verifying

Right-click in the Chat view → **Diagnostics** to see every loaded agent and any
frontmatter errors. Hovering an agent in **Configure Custom Agents** shows its
source path.
