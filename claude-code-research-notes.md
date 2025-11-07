# Claude Code Research - Sticky Notes

**Context-light, insight-rich reference guide**

---

## 🔌 PLUGINS

**What:** Extensions adding custom functionality to Claude Code

**5 Component Types:**
1. **Commands** - User-triggered slash commands (Markdown)
2. **Agents** - Context-aware subagents (auto-invoked)
3. **Skills** - Model-invoked capabilities (Claude chooses)
4. **Hooks** - Event handlers for automation
5. **MCP Servers** - External tool integrations

**Structure:**
```
plugin-name/
├── .claude-plugin/plugin.json  # REQUIRED manifest
├── commands/                   # Slash commands
├── agents/                     # Subagent definitions
├── skills/                     # Skill definitions
├── hooks/hooks.json           # Event handlers
└── .mcp.json                  # MCP config
```

**⚠️ Critical:** Components MUST be at plugin root, NOT inside `.claude-plugin/`

**Install:** `/plugin install plugin-name@marketplace-name`

**Management:** `/plugin` for interactive menu

**Team Setup:** Use `.claude/settings.json` at repo level for auto-install

**Gotcha:** Restart required after installation

---

## 🎯 SKILLS

**What:** Model-invoked capabilities Claude uses autonomously

**Key Distinction:** Claude decides WHEN to use (not user-triggered)

**Storage:**
- Personal: `~/.claude/skills/`
- Project: `.claude/skills/`
- Plugins: Bundled in marketplace packages

**Minimum Structure:**
```
skill-name/
└── SKILL.md (required)
```

**Required Frontmatter:**
```yaml
---
name: skill-name
description: What it does AND when to invoke it
allowed-tools: Read, Grep, Glob  # Optional
---
```

**⚠️ Critical:** Description MUST explain BOTH capability AND trigger conditions

**Naming Rules:**
- Lowercase, hyphens only
- Max 64 chars
- Example: `pdf-analyzer`, `api-docs-generator`

**Discovery:** Ask "What Skills are available?"

**Gotcha:** Changes require Claude Code restart

**Tool Restrictions:** `allowed-tools` only works in Claude Code (not other Claude products)

---

## 🤖 SUB-AGENTS

**What:** Specialized AI assistants with independent context windows

**Key Benefit:** Task delegation without polluting main conversation

**Storage:**
- Project: `.claude/agents/`
- User: `~/.claude/agents/`
- CLI: `--agents` flag

**Structure (Markdown with YAML):**
```yaml
---
name: agent-name
description: When to use this agent
tools: Tool1, Tool2        # Optional: inherits all if omitted
model: sonnet              # Optional: sonnet/opus/haiku
---

System prompt defining role and approach...
```

**⚠️ Critical Limitations:**
1. Cannot spawn other sub-agents (no nesting)
2. Separate context = must re-gather info each time
3. Omitting `tools` = inherits ALL tools (security risk)

**Management:** `/agents` command for interactive menu

**Resume:** Each execution gets unique `agentId`, can resume with transcripts

**Best Practice:** Single responsibility principle - one clear purpose per agent

---

## ⚡ SLASH COMMANDS

**What:** Interactive prompts from Markdown files

**4 Types:**
1. Built-in (`/clear`, `/model`, `/help`)
2. Custom (user-defined `.md` files)
3. Plugin (auto-registered from plugins)
4. MCP (format: `/mcp__<server>__<prompt>`)

**Storage:**
- Project: `.claude/commands/`
- Personal: `~/.claude/commands/`

**Basic Structure:**
```markdown
---
description: Shows in /help (REQUIRED for tool invocation)
allowed-tools: [Bash, Read, Write]
model: claude-sonnet-4-5
argument-hint: <file-path>
disable-model-invocation: true  # Prevent auto-invocation
---

Your prompt here.
Use $ARGUMENTS or $1, $2 for params.
Reference files with @src/file.js
```

**Dynamic Features:**
- `$ARGUMENTS` = all args
- `$1`, `$2` = positional params
- `@path/to/file` = includes file content
- `!command` = bash execution (requires `allowed-tools: [Bash]`)

**⚠️ Critical:**
- Must have `description` for Claude to invoke via SlashCommand tool
- `allowed-tools: [Bash]` MANDATORY for `!` prefix
- 15,000 char metadata limit

**vs Skills:** Commands = user-invoked, Skills = Claude-invoked

---

## 🔗 MCP (Model Context Protocol)

**What:** Open standard for connecting Claude to external tools/services

**3 Transport Types:**
1. **HTTP** (Recommended) - Cloud services
2. **Stdio** - Local processes
3. **SSE** (Deprecated) - Use HTTP instead

**Add Server:**
```bash
# HTTP
claude mcp add --transport http notion https://mcp.notion.com/mcp

# Stdio (note the -- separator)
claude mcp add --transport stdio airtable \
  --env KEY=value -- npx -y airtable-mcp-server
```

**⚠️ CRITICAL:** Double-dash `--` separates Claude CLI flags from server commands

**Scopes:**
- Local (default) - Private to directory
- Project - Shared via `.mcp.json` in git
- User - Across all projects

**Management:**
```bash
claude mcp list
claude mcp remove server-name
/mcp  # Check status in session
```

**Security Warnings:**
- Third-party servers NOT verified by Anthropic
- Prompt injection risks with untrusted content
- Output limit: 25,000 tokens (adjustable via `MAX_MCP_OUTPUT_TOKENS`)

**Enterprise:** Admins can deploy `managed-mcp.json` to control access

---

## ⚙️ SETTINGS & CONFIGURATION

**Hierarchy (highest to lowest):**
1. Enterprise managed policies
2. Command-line arguments
3. Project local (`.claude/settings.local.json`)
4. Project shared (`.claude/settings.json`)
5. User (`~/.claude/settings.json`)

**Key Settings:**

| Setting | Purpose |
|---------|---------|
| `permissions` | Tool access control (allow/ask/deny) |
| `env` | Environment variables |
| `model` | Override default model |
| `apiKeyHelper` | Script for auth credentials |
| `cleanupPeriodDays` | Transcript retention (default: 30) |
| `includeCoAuthoredBy` | Claude git commit attribution |
| `outputStyle` | System prompt behavior |
| `forceLoginMethod` | Restrict to `claudeai` or `console` |

**Permission System:**
- `allow`: Explicit permission (no prompt)
- `ask`: Confirmation prompt
- `deny`: Block completely

**Pattern Matching:** Prefix-based (not regex)
```json
"allow": ["Bash(npm run test:*)"]
```

**Tools Requiring Permissions:**
- Bash, Edit, SlashCommand, WebFetch, WebSearch, Write

**Tools Without Permissions:**
- Glob, Grep, Read, NotebookRead, Task, TodoWrite

**Sandbox:**
- macOS/Linux only
- `sandbox.enabled: true`
- `autoAllowBashIfSandboxed: true`

**⚠️ Gotcha:** Prefix matching can be bypassed with creative commands

---

## 🪝 HOOKS

**What:** Automated shell commands triggered by Claude Code events

**9 Event Types:**
- PreToolUse - Before tool execution (can block)
- PostToolUse - After tool completes
- UserPromptSubmit - Before Claude processes
- Notification - When notifying user
- Stop - After main agent finishes
- SubagentStop - After subagent finishes
- PreCompact - Before compact operations
- SessionStart - New/resumed sessions
- SessionEnd - Session termination

**Structure:**
```json
{
  "hooks": {
    "EventName": [{
      "matcher": "ToolName",  # Exact, regex, or *
      "hooks": [{
        "type": "command",
        "command": "script"
      }]
    }]
  }
}
```

**Exit Codes:**
- **0**: Success (stdout shown to user)
- **2**: Blocking error (stderr to Claude)
- **Other**: Non-blocking error (stderr to user)

**JSON Output:**
```json
{
  "continue": false,
  "stopReason": "Message",
  "suppressOutput": true,
  "systemMessage": "Warning"
}
```

**Environment Variables:**
- `$CLAUDE_PROJECT_DIR`
- `$CLAUDE_ENV_FILE` (SessionStart only)
- `$CLAUDE_CODE_REMOTE`
- `${CLAUDE_PLUGIN_ROOT}`

**⚠️ CRITICAL SECURITY:**
- Hooks run with YOUR credentials automatically
- Can cause data loss if malicious/buggy
- Always quote variables: `"$VAR"` not `$VAR`
- Validate all inputs

**Gotcha:** Config read at startup - external edits require restart

**Use Cases:**
- Automatic formatting (prettier, gofmt)
- File protection (.env, secrets)
- Command logging
- Desktop notifications

---

## 🧠 MEMORY (CLAUDE.md)

**What:** Hierarchical markdown-based persistent instructions

**4-Level Hierarchy:**
1. **Enterprise** - Organization-wide
2. **Project** - `./CLAUDE.md` or `./.claude/CLAUDE.md`
3. **User** - `~/.claude/CLAUDE.md`
4. **Project Local** - `./CLAUDE.local.md` (DEPRECATED)

**Loading:** Recursive upward search from current directory, stops before root

**Precedence:** Higher hierarchy files loaded FIRST (take precedence)

**Import Syntax:** `@path/to/file`
- Supports relative/absolute paths
- Home directory: `@~/.claude/personal.md`
- Max depth: 5 hops
- Ignored inside code spans

**Commands:**
- `#` - Quick add with file selection
- `/memory` - Open in editor
- `/init` - Bootstrap project CLAUDE.md

**Best Practices:**
- Be specific, avoid vague instructions
- Use markdown headings for organization
- Personal prefs via imports (not version control)
- Regular review/audit

**⚠️ Gotcha:** Subdirectory memories discovered only when accessing those files

---

## 💻 CLI REFERENCE

**Core Commands:**
```bash
claude                    # Interactive REPL
claude "query"           # REPL with initial prompt
claude -p "query"        # Non-interactive, print & exit
cat file | claude -p "q" # Process stdin
claude -c                # Resume last conversation
claude update            # Update CLI
claude mcp               # Configure MCP servers
```

**Key Flags:**

**Output Control:**
- `--print/-p` - Non-interactive mode
- `--output-format <format>` - text/json/stream-json
- `--include-partial-messages` - Streaming events

**System Prompt:**
- `--system-prompt <text>` - REPLACE entire prompt
- `--system-prompt-file <path>` - Load from file
- `--append-system-prompt <text>` - ADD to prompt (recommended)

**Permissions:**
- `--allowedTools <tools>` - Whitelist
- `--disallowedTools <tools>` - Blacklist
- `--permission-mode <mode>` - Permission handling
- `--dangerously-skip-permissions` - Bypass all (use carefully)

**Execution:**
- `--model <model>` - Set model
- `--max-turns <number>` - Limit agentic iterations
- `--verbose` - Debug output
- `--add-dir <path>` - Additional working directories

**⚠️ Critical:**
- `--system-prompt` removes Claude Code's default behaviors
- Use `--append-system-prompt` to preserve built-in capabilities
- `--system-prompt-file` requires `-p` flag

---

## 🖥️ INTERACTIVE MODE

**What:** Primary CLI interface with specialized input methods

**Input Navigation:**
- Ctrl+R - Reverse command history search
- Ctrl+C - Cancel input/generation
- Esc+Esc - Rewind code/conversation
- Ctrl+B - Background long-running command (Ctrl+B twice in tmux)

**Special Prefixes:**
- `#` - Add to CLAUDE.md memory
- `/` - Execute slash command
- `!` - Direct bash execution
- `@` - File path autocomplete

**Multiline Input:**
- Backslash + Enter (default)
- Option + Enter (macOS)
- Shift + Enter (requires terminal setup)
- Ctrl + J (universal fallback)

**Enhanced Features:**
- Tab - Toggle extended thinking
- Ctrl+V - Paste images (Alt+V on Windows)

**Background Processing:**
- Async execution for build tools, test runners, dev servers
- Output buffered and retrievable via BashOutput tool

**⚠️ Gotcha:**
- `!` history expansion disabled by default
- Tmux users press Ctrl+B twice

---

## 🎛️ MODEL CONFIGURATION

**Model Aliases:**
- `default` - Account-dependent
- `sonnet` - Claude Sonnet 4.5 (balanced)
- `opus` - Claude Opus 4.1 (complex reasoning)
- `haiku` - Claude Haiku 4.5 (fast)
- `sonnet[1m]` - 1M context window
- `opusplan` - Opus plans, Sonnet executes (hybrid)

**Configuration Priority:**
1. `/model <alias>` - Mid-session command
2. `claude --model <alias>` - Startup flag
3. `ANTHROPIC_MODEL` - Environment variable
4. Settings file - Permanent config

**Environment Variables:**
```bash
ANTHROPIC_DEFAULT_OPUS_MODEL=...
ANTHROPIC_DEFAULT_SONNET_MODEL=...
ANTHROPIC_DEFAULT_HAIKU_MODEL=...
CLAUDE_CODE_SUBAGENT_MODEL=...
```

**Prompt Caching:**
- Enabled by default
- Disable: `DISABLE_PROMPT_CACHING=1`
- Per-model: `DISABLE_PROMPT_CACHING_HAIKU=1`

**⚠️ Gotchas:**
- Aliases auto-update to latest versions
- Pin to full model names for version consistency
- Hybrid mode switches models mid-workflow

---

## 🚀 HEADLESS MODE

**What:** Non-interactive execution for automation/CI/CD

**Trigger:** `-p` or `--print` flag

**Input Methods:**
- Direct: `claude -p "query"`
- Stdin: `echo "query" | claude -p`
- JSONL streaming: Multi-turn via structured input

**Output Formats:**
- `text` - Plain text
- `json` - Structured with metadata
- `stream-json` - Real-time message stream

**JSON Structure:**
```json
{
  "cost": "0.0234",
  "duration": "15.2s",
  "turns": 3,
  "sessionId": "abc123",
  "error": false,
  "response": "..."
}
```

**Essential Flags:**
- `-p, --print` - Enable headless
- `--output-format <format>` - Control output
- `--resume <session-id>` - Continue conversation
- `--allowedTools <list>` - Whitelist tools
- `--append-system-prompt <text>` - Custom instructions
- `--verbose` - Debug logging

**⚠️ Critical:**
- No built-in rate limiting (implement delays)
- Must capture session IDs for multi-turn
- Always check exit codes
- Some interactive tools may not work

**Use Cases:**
- CI/CD automation
- Batch processing
- Incident response
- Scheduled tasks

---

## 💾 CHECKPOINTING

**What:** Automatic snapshots before Claude edits

**Behavior:**
- New checkpoint per user prompt
- Persists across resumed conversations
- Auto-deleted after 30 days (customizable)

**Rewind:** Esc+Esc or `/rewind`
- Conversation only
- Code only
- Both

**⚠️ CRITICAL LIMITATION:**
- Only tracks Claude's direct edits
- **Does NOT track bash operations** (rm, mv, cp, etc.)
- External edits not captured
- **NOT a replacement for Git**

**Use Cases:**
- Safe experimentation
- Quick bug fix rollback
- Feature exploration

---

## 📦 INSTALLATION & SETUP

**System Requirements:**
- OS: macOS 10.15+, Ubuntu 20.04+, Windows 10+ (WSL/Git Bash)
- RAM: 4GB minimum
- Node.js 18+ (NPM install only)
- Shell: Bash, Zsh, or Fish

**Installation:**
```bash
# Native (Recommended)
curl -fsSL https://claude.ai/install.sh | bash

# Windows PowerShell
irm https://claude.ai/install.ps1 | iex

# NPM
npm install -g @anthropic-ai/claude-code
```

**⚠️ CRITICAL:** NEVER use `sudo npm install -g` (security risk)

**Verification:**
```bash
claude doctor
```

**Authentication:**
- Claude Console (billing account)
- Claude Pro/Max subscription
- Enterprise: Bedrock/Vertex AI

**Launch:**
```bash
cd your-project
claude
```

**Gotchas:**
- Requires internet connection
- Must be in supported country
- Alpine Linux needs manual library install
- Claude Code workspace dedicated exclusively to CLI

---

## 🗂️ COMMON WORKFLOWS

**Plan Mode:**
- Shift+Tab to toggle
- Read-only analysis
- CLI: `claude --permission-mode plan`

**Extended Thinking:**
- Tab key to enable
- Deep reasoning for complex problems
- Disabled by default

**File References:**
- `@file/path` for explicit inclusion
- `@server:resource` for MCP resources

**Image Analysis:**
- Drag/drop, Ctrl+V paste, or file path
- Use cases: Screenshots, diagrams, mockups

**Unix-style Integration:**
```bash
# Headless
cat file.txt | claude -p 'prompt' > output.txt

# Resume
claude --continue --print "next feature"
```

**Best Practices:**
- Start broad → narrow to specifics
- Request multiple solutions before choosing
- Make incremental testable changes
- Use Plan Mode before complex changes
- Create project-specific agents for team consistency

---

## 🎨 OUTPUT STYLES (DEPRECATED)

**⚠️ REMOVED: November 5, 2025**

**Migration:**
- Use `--system-prompt-file <path>` instead
- Use `--append-system-prompt` to extend default
- Use `explanatory-output-style` plugin for educational mode

**Old Built-in Styles:**
- Default - Task-focused engineering
- Explanatory - Educational with insights
- Learning - Collaborative with TODO(human) markers

---

## 🔑 KEY INSIGHTS

**Plugin Architecture:**
- 5 component types (commands, agents, skills, hooks, MCP)
- Components at plugin root, NOT in `.claude-plugin/`
- Restart required after installation

**Invocation Models:**
- Commands: User-triggered (`/command`)
- Skills: Claude-invoked (autonomous)
- Agents: Context-based delegation
- Hooks: Event-triggered automation

**Security Layers:**
- Permission system (allow/ask/deny)
- Sandbox mode (macOS/Linux)
- Tool restrictions per component
- Hook security critical (runs with your credentials)

**Context Management:**
- CLAUDE.md: Persistent memory hierarchy
- Sub-agents: Independent context windows
- Checkpointing: Session-level safety net
- MCP: External data integration

**Automation:**
- Headless mode: CI/CD integration
- Hooks: Event-driven automation
- Background processing: Async execution
- JSONL streaming: Multi-turn conversations

**Configuration Hierarchy:**
Enterprise > CLI flags > Project local > Project shared > User

**Model Strategy:**
- Sonnet: Daily tasks
- Opus: Complex reasoning
- Haiku: Fast/simple
- Aliases auto-update to latest

---

## 📚 QUICK REFERENCE

**Essential Commands:**
```bash
/help              # Show available commands
/agents            # Manage sub-agents
/plugin            # Plugin management
/model <alias>     # Switch model
/memory            # Edit memory files
/rewind            # Restore previous state
/clear             # Clear conversation
/vim               # Enable vim mode
```

**File Locations:**
```
~/.claude/                      # User-level config
  ├── CLAUDE.md                # User memory
  ├── settings.json            # User settings
  ├── agents/                  # User sub-agents
  ├── commands/                # User slash commands
  └── skills/                  # User skills

.claude/                        # Project-level config
  ├── CLAUDE.md                # Project memory
  ├── settings.json            # Shared settings
  ├── settings.local.json      # Personal overrides
  ├── agents/                  # Project sub-agents
  ├── commands/                # Project slash commands
  └── skills/                  # Project skills

plugin-name/
  ├── .claude-plugin/
  │   └── plugin.json          # Plugin manifest
  ├── commands/
  ├── agents/
  ├── skills/
  ├── hooks/
  │   └── hooks.json
  └── .mcp.json
```

**Environment Variables:**
```bash
# Model Configuration
ANTHROPIC_MODEL=claude-sonnet-4-5-20250929
ANTHROPIC_API_KEY=sk-...
CLAUDE_CODE_SUBAGENT_MODEL=...

# Feature Control
MAX_THINKING_TOKENS=10000
DISABLE_TELEMETRY=1
DISABLE_ERROR_REPORTING=1
DISABLE_PROMPT_CACHING=1
DISABLE_AUTOUPDATER=1

# Providers
CLAUDE_CODE_USE_BEDROCK=1
CLAUDE_CODE_USE_VERTEX=1

# Limits
BASH_MAX_TIMEOUT_MS=120000
BASH_MAX_OUTPUT_LENGTH=30000
MAX_MCP_OUTPUT_TOKENS=25000
```

**Permission Patterns:**
```json
{
  "permissions": {
    "allow": [
      "Bash(npm run test:*)",
      "Edit(src/**)"
    ],
    "ask": [
      "WebFetch(*)"
    ],
    "deny": [
      "Edit(.env*)",
      "Edit(secrets/**)"
    ]
  }
}
```

---

**Last Updated:** 2025-01-04
**Source:** https://docs.claude.com/en/docs/claude-code/
**Research Method:** Systematic single-page delegation to claude-code-expert sub-agents
