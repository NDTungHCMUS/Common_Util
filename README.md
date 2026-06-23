# Claude Code Project Template

A reusable starter that wires [CodeGraph](https://github.com/colbymchenry/codegraph)
(code-intelligence MCP server) plus a curated set of Claude Code commands, skills,
and agents into any project.

## What's included

| Path | Purpose |
|------|---------|
| `.mcp.json` | Registers the `codegraph` MCP server (stdio) for this project. |
| `.claude/settings.json` | Shared, committed settings — pre-approves the codegraph server and auto-allows its tools (no prompts). |
| `.claude/commands/` | Slash commands. |
| `.claude/skills/` | Skills. |
| `.claude/agents/` | Agent definitions. |

Not committed (gitignored): `.claude/settings.local.json` (personal/machine-local)
and `.codegraph/` (the local index DB).

## Prerequisite — once per machine

The `.mcp.json` launches `codegraph`, so the CLI must be on your PATH:

```bash
npm i -g @colbymchenry/codegraph
```

Do this once per computer, not per project.

## Use in a new project

**Start a fresh project from this template:**

```bash
npx degit <your-user>/<this-repo> my-new-project
cd my-new-project
git init
codegraph init        # builds .codegraph/ for this project
```

**Or add it to an existing project:**

```bash
git clone --depth 1 <this-repo-url> /tmp/tpl
cp -r /tmp/tpl/.claude .
cp /tmp/tpl/.mcp.json .
cat /tmp/tpl/.gitignore >> .gitignore
rm -rf /tmp/tpl
codegraph init
```

(If the project already has `.claude/settings.json`, merge by hand rather than overwriting.)

Then **restart Claude Code** — MCP servers load at session start, so codegraph
appears in a fresh session. Auto-sync keeps the index fresh as you edit code.
