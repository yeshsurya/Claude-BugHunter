---
title: Multi-harness install
nav_order: 3
description: Run the Claude-BugHunter skills on GitHub Copilot and other agent harnesses.
---

# Multi-harness install

The 85 skills are plain **Agent Skills** (`SKILL.md` = `name` + `description` frontmatter + Markdown). That format is an open standard, so the *knowledge* runs on more than Claude Code. This page also covers **GitHub Copilot CLI, the Copilot app, and VS Code**, **OpenCode**, **OpenAI Codex CLI**, **Hermes Agent**, and **Google AntiGravity**.

> **What ports and what doesn't.** The **85 skills** (payloads, methodology, bypass tables, disclosed-report patterns) port to every harness below. The **`/hunt` slash commands, the plugin marketplace, and the `hunt-dispatch` subagent routing are Claude-Code-specific** and do **not** port — other harnesses get the knowledge, not the orchestration engine. **Burp MCP** ports to all of them (it's just an MCP server).

## Compatibility matrix (verified mid-2026)

| Harness | Reads `SKILL.md`? | Skill path it loads | MCP (Burp) | Slash commands |
|---|---|---|---|---|
| **Claude Code** (baseline) | ✅ native | `~/.claude/skills/` | ✅ | ✅ (`/hunt`, …) |
| **GitHub Copilot CLI, app, VS Code** | ✅ native | `~/.agents/skills/` | Separate setup; HackerOne not installed | Claude commands not installed |
| **OpenCode** | ✅ native | reads `~/.claude/skills/` **and** `~/.agents/skills/` | ✅ `opencode.json` | ✅ own format |
| **Codex CLI** | ✅ native | `~/.agents/skills/` (does *not* read `~/.claude/`) | ✅ `~/.codex/config.toml` | ✅ own format |
| **Hermes Agent** | ✅ (agentskills.io) | `~/.hermes/skills/` | ✅ | ✅ own format |
| **Google AntiGravity** | ✅ native | `~/.gemini/config/skills/` | ✅ `~/.gemini/config/mcp_config.json` | ✅ own format |

**Key:** `~/.agents/skills/` is the shared path read by **Copilot, Codex and OpenCode**. The other paths are `~/.claude/skills/`, `~/.hermes/skills/`, and `~/.gemini/config/skills/`. The installer shortens overlong descriptions in the shared copy.

## Install

One command installs the skills to every harness's path (copy install; existing skills are backed up outside the loading path):

```bash
# macOS / Linux
git clone https://github.com/elementalsouls/Claude-BugHunter.git
cd Claude-BugHunter
bash scripts/install.sh --all          # Claude + ~/.agents/skills (Codex/OpenCode) + ~/.hermes/skills + ~/.gemini/config/skills
```

```powershell
# Windows (PowerShell)
git clone https://github.com/elementalsouls/Claude-BugHunter.git
cd Claude-BugHunter
pwsh ./scripts/install.ps1 -All         # Claude + ~/.agents/skills (Codex/OpenCode) + ~/.hermes/skills + ~/.gemini/config/skills
```

Pick specific harnesses instead:

```bash
# macOS / Linux
bash scripts/install.sh                 # Claude Code only (default)
bash scripts/install.sh --agents        # + Codex & OpenCode (~/.agents/skills)
bash scripts/install.sh --copilot       # + Copilot CLI, app & VS Code (~/.agents/skills)
bash scripts/install.sh --hermes        # + Hermes Agent (~/.hermes/skills)
bash scripts/install.sh --antigravity   # + Google AntiGravity (~/.gemini/config/skills)
```

```powershell
# Windows (PowerShell)
pwsh ./scripts/install.ps1              # Claude Code only (default)
pwsh ./scripts/install.ps1 -Agents      # + Codex & OpenCode (~/.agents/skills)
pwsh ./scripts/install.ps1 -Copilot     # + Copilot CLI, app & VS Code (~/.agents/skills)
pwsh ./scripts/install.ps1 -Hermes      # + Hermes Agent (~/.hermes/skills)
pwsh ./scripts/install.ps1 -AntiGravity # + Google AntiGravity (~/.gemini/config/skills)
```

- **OpenCode** already reads `~/.claude/skills/`, so the plain installer (no flags) is enough for OpenCode — you don't need `--agents`/`-Agents` for it. That flag exists mainly for **Codex** (which reads only `~/.agents/skills/`).
  - *Caveat (verified):* OpenCode reads **both** `~/.claude/skills/` and `~/.agents/skills/`. If both are populated, OpenCode logs harmless `duplicate skill name` warnings and loads one copy.
- **Codex is the strict parser** (verified by testing): it hard-rejects descriptions > 1024 chars and invalid YAML, where Claude/OpenCode/Hermes/AntiGravity are lenient. So the installer **auto-truncates** any description > 1024 to ≤1024 **only in the `~/.agents/skills` (Codex) copy** — your `~/.claude`, `~/.hermes`, and `~/.gemini/config/skills` copies keep the full descriptions (incl. non-English trigger words). The install logs which were truncated (today: the 3 aggregator router skills). `--normalize-frontmatter` (`-NormalizeFrontmatter`) additionally strips the non-standard `sources:`/`report_count:` keys (optional — Codex tolerates them).

## GitHub Copilot and codebase privacy

Use `--copilot` / `-Copilot` even if only VS Code or the app is installed:
`--all` detects the Copilot CLI or its `~/.copilot` directory, not every VS Code
or app installation. It also installs the usual Claude files. The shared
`~/.agents/skills/` copy is available to Copilot CLI, the app, and VS Code
agent sessions; avoid installing a second copy in another skill directory.
Use Copilot's skill list or VS Code's Agent Customizations view to verify
discovery. Skill selection is contextual, not guaranteed. The deterministic
`cbh` CLI remains available separately; Claude commands and the headless
Claude-only engine are not automatically ported.

**HackerOne is not configured for Copilot by this installer**, even with
`--hackerone-mcp` / `-HackeroneMcp`; that flag may still configure other selected
harnesses. Existing Copilot or VS Code MCP settings are not changed. Before
reviewing private code, disable any existing HackerOne server in each client:
for CLI use `copilot mcp disable hackerone` (substitute its configured name),
and check the app and VS Code MCP management views as well. If a server is
defined in another configuration source, disable or remove it there. The app
can share CLI settings; verify its effective server list rather than assuming.

The bundle's workflow skills and HackerOne tool descriptions instruct agents
not to send codebase details to HackerOne or other external research services.
These are instructions, **not an enforceable data-loss boundary**: other skills,
shell commands, web tools, or already-configured MCP servers may make network
requests. Restrict network access and tool permissions when confidentiality
requires enforcement. Copilot and the configured model provider may still
process code provided to them under their own data policies.

## Burp MCP on other harnesses

Your Burp MCP is a stdio command. `--burp-mcp` (`-BurpMcp`, with a supported harness flag) translates your **existing** Claude Code Burp definition (from `~/.claude.json`) and backs up each config first. Copilot is not configured by this flag.

```bash
# macOS / Linux
bash scripts/install.sh --all --burp-mcp     # writes OpenCode, Codex, and AntiGravity MCP config; prints Hermes guidance
```

```powershell
# Windows (PowerShell)
pwsh ./scripts/install.ps1 -All -BurpMcp      # writes OpenCode, Codex, and AntiGravity MCP config; prints Hermes guidance
```

Or do it manually (replace the jar path / port with yours):

**OpenCode** — `~/.config/opencode/opencode.json`
```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "burp": {
      "type": "local",
      "command": ["java", "-jar", "~/.BurpSuite/mcp-proxy/mcp-proxy-all.jar", "--sse-url", "http://127.0.0.1:9876"],
      "enabled": true
    }
  }
}
```

**Codex** — `~/.codex/config.toml`
```toml
[mcp_servers.burp]
command = "java"
args = ["-jar", "~/.BurpSuite/mcp-proxy/mcp-proxy-all.jar", "--sse-url", "http://127.0.0.1:9876"]
```

**Google AntiGravity** — `~/.gemini/config/mcp_config.json`
```json
{
  "mcpServers": {
    "burp": {
      "command": "java",
      "args": ["-jar", "~/.BurpSuite/mcp-proxy/mcp-proxy-all.jar", "--sse-url", "http://127.0.0.1:9876"]
    }
  }
}
```

**Hermes** — see the [Hermes MCP guide](https://hermes-agent.nousresearch.com/docs/guides/use-mcp-with-hermes); use the same `java -jar … --sse-url …` command.

## HackerOne MCP on other harnesses

The in-repo HackerOne MCP (`mcp/hackerone/server.py`, stdlib-only stdio) also translates 1:1. `--hackerone-mcp` wires it automatically (public tools need no key; set `HACKERONE_API_USERNAME` / `HACKERONE_API_TOKEN` before running to carry the logged-in tools into the config):

```bash
# macOS / Linux — wire both MCPs into supported detected harnesses, excluding Copilot
bash scripts/install.sh --all --burp-mcp --hackerone-mcp
```

Or manually (replace `/path/to/Claude-BugHunter` with your clone):

**OpenCode** — `~/.config/opencode/opencode.json`
```json
{
  "mcp": {
    "hackerone": {
      "type": "local",
      "command": ["python3", "/path/to/Claude-BugHunter/mcp/hackerone/server.py"],
      "enabled": true
    }
  }
}
```

**Codex** — `~/.codex/config.toml`
```toml
[mcp_servers.hackerone]
command = "python3"
args = ["/path/to/Claude-BugHunter/mcp/hackerone/server.py"]
```

**Google AntiGravity** — `~/.gemini/config/mcp_config.json`
```json
{
  "mcpServers": {
    "hackerone": {
      "command": "python3",
      "args": ["/path/to/Claude-BugHunter/mcp/hackerone/server.py"]
    }
  }
}
```

**Hermes** — same stdio command under the `mcp_servers:` block; see the Hermes MCP guide. On Windows give the full path to `python.exe` (`python3` there is often the Store stub).

## Verify it loaded
- **OpenCode / Codex / Hermes / Google AntiGravity:** open the tool and describe a task (e.g. *"test this endpoint for SSRF"*) — the matching `hunt-*` skill should auto-load by its description, same as in Claude Code.
- **Hermes:** `hermes skills` should list the bundle from `~/.hermes/skills/`.
