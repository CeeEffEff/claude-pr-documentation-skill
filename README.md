# PR Documentation Generator

A Claude Code skill that automates comprehensive documentation for Pull Requests using graph-based analysis. This tool analyzes PR changes, categorizes them by impact, extracts entities and relationships, and generates structured technical documentation.

## Overview

This project provides a `/pr-doc` slash command that orchestrates file analysis through specialized agents, stores findings in a Neo4j graph database, and generates detailed markdown documentation covering technical changes, breaking changes, and architectural impacts.

## Architecture

### Claude Code Components

**Slash Command**: `/pr-doc <pr-number> [repo-path]`
- Entry point for generating PR documentation
- Located in `.claude/commands/pr-doc.md`
- Orchestrates the entire workflow from PR metadata retrieval to documentation generation

**Agents**:
- `file-analyzer`: Specialized agent that analyzes individual files in isolation
  - Examines before/after states using git
  - Extracts entities (functions, classes, services, dependencies)
  - Stores structured findings in Neo4j
  - Operates in isolated context to prevent context pollution

**Skills**:
- `file-pattern-detector`: Detects file types and extracts structural metadata
  - Classifies files (React components, API endpoints, configs, tests, etc.)
  - Identifies frameworks (Next.js, Express, Terraform)
  - Extracts imports, exports, functions, classes

- `neo4j-operations`: Handles all Neo4j graph database interactions
  - Creates entities following standardized schema
  - Establishes relationships between entities
  - Queries graph for architectural insights
  - Validates operations against schema

### Technology Stack

- **Claude Code**: Orchestration, agents, and skills framework
- **GitHub CLI** (`gh`): PR metadata and file diff retrieval
- **Neo4j** (via MCP): Graph database for relationship tracking
- **MCP Tools**: `mcp__mcp-neo4j-memory__*` for entity-observation model
- **Git**: File history and diff analysis

## How It Works

### Workflow Sequence

1. **PR Metadata Retrieval**
   - Uses `gh pr view` to fetch PR details (title, author, files, commits)
   - Stores context in `memory_bank/prContext.md`
   - Creates PR entity in Neo4j: `pr:{number}`

2. **File Analysis Strategy**
   - **Threshold**: PRs with <12 effective files analyze all; ≥12 files use strategic sampling
   - **Effective Count**: Excludes generated files, lock files, trivial updates
   - **Delegation**: Each file analyzed by `file-analyzer` agent in isolation

3. **Per-File Analysis** (via `file-analyzer` agent)
   - Reads file's "before" state: `git show {base_commit}:{file_path}`
   - Detects file type using `file-pattern-detector` skill
   - Reads changes: `git diff {base_commit}..{head_commit}`
   - Extracts entities: functions, classes, business logic, services, dependencies
   - Analyzes impact: lines added/removed, change type, impact level
   - Stores in Neo4j using `neo4j-operations` skill
   - Saves JSON results to `memory_bank/analysisResults/{file_hash}.json`

4. **Graph-Based Representation**
   - Entities: `PR`, `File`, `Change`, `Function`, `Class`, `BusinessLogic`, `Configuration`, `Service`, `Dependency`
   - Relationships: `MODIFIES`, `HAS_CHANGE`, `CONTAINS`, `DEPENDS_ON`, `IMPLEMENTS`
   - Entity naming: `pr:{number}`, `file:{path}`, `bl:{name}`, `config:{name}`, `service:{name}`
   - Schema reference: `memory_bank/graphSchema.md`

5. **Documentation Generation**
   - Reads ALL JSON files from `memory_bank/analysisResults/*.json`
   - Queries Neo4j for architectural insights and relationships
   - Generates structured markdown with three parts:
     - **Executive Summary**: Overview, statistics, categorization
     - **Technical Deep-Dive**: File-by-file analysis with breaking changes, entities, impact
     - **Cross-Cutting Concerns**: Architecture, dependencies, testing recommendations
   - Output: `pr-{number}-documentation.md`

## Neo4j Entity-Observation Model

This project uses the Neo4j MCP's entity-observation model instead of traditional graph properties:

- **Entities**: Named nodes with type classifications (e.g., `pr:301`, `file:src/api.ts`)
- **Observations**: Atomic facts stored as text arrays (e.g., "Lines added: 42", "Impact level: high")
- **Relations**: Directed connections with active-voice types (e.g., `MODIFIES`, `DEPENDS_ON`)

This approach enables flexible, incremental knowledge building without rigid schema constraints.

## Key Features

### Context Management
- Explicit handling of large PRs with >10 files
- Agent-based isolation prevents context pollution
- Continuation tasks for PRs approaching context limits
- JSON file persistence for analysis results

### File Categorization
- Business logic, configuration, services, dependencies
- Schemas, infrastructure, CI/CD pipelines
- Focus on purpose, behavior, and system impact

### Breaking Change Detection
- Extracted per-file and documented explicitly
- Method signature changes, API modifications
- Dependency version changes

### Architectural Insights
- Graph queries reveal service dependencies
- Cross-file impact analysis
- Relationship tracking between components

## Project Structure

```
.claude/
├── commands/
│   └── pr-doc.md              # Main slash command
├── agents/
│   └── file-analyzer.md       # File analysis agent
├── skills/
│   ├── file-pattern-detector/ # File type detection skill
│   │   └── SKILL.md
│   └── neo4j-operations/      # Graph database skill
│       └── SKILL.md
└── CLAUDE.md                  # Project-level instructions

memory_bank/
├── prContext.md               # Current PR workflow state
├── graphSchema.md             # Neo4j entity/relation schema
├── analysisResults/           # Per-file analysis JSON
└── [other context files]
```

## Usage

```bash
# Generate documentation for PR #123 in current directory
/pr-doc 123

# Generate documentation for PR #456 in specific repo
/pr-doc 456 /path/to/repo
```

The command will:
1. Fetch PR metadata from GitHub
2. Analyze changed files using specialized agents
3. Store findings in Neo4j graph database
4. Generate comprehensive markdown documentation

## Example Output

```markdown
# PR #301: Add Dark Mode Support

## Executive Summary
- **Author**: john-doe
- **Files Changed**: 8
- **Impact**: High - Breaking changes to theme API

## Technical Deep-Dive

### src/themes/darkMode.ts
**Changes**: +274/-192 lines | **Impact**: HIGH

**Breaking Changes**:
1. Removed `ThemeInterface` inheritance
2. `applyTheme()` signature changed - removed `cascade` parameter

**Key Changes**:
- Migrated from CSS-in-JS to CSS variables
- Added `getThemePreference()` for system detection

**Entities Modified**:
- **Classes**: DarkModeManager (refactored)
- **Functions**: applyTheme, getThemePreference
- **Dependencies Added**: @theme-utils/css-vars

**Impact**: Affects all theme consumers. Migration required.

## Cross-Cutting Concerns
- Service dependencies: ThemeService, UserPreferencesService
- Testing: Add dark mode E2E tests
- Deployment: Update CSS build pipeline
```

## Contributing

1. Review workflow structure in `.claude/commands/pr-doc.md`
2. Extend `file-analyzer` agent for new file types
3. Update `graphSchema.md` for new entity types
4. Enhance `file-pattern-detector` for framework detection
5. Test with various PR sizes and types

## Technical Design Decisions

**Why Agents?**: File analysis in isolated contexts prevents context pollution and enables parallel processing patterns in future iterations.

**Why Graph Database?**: Relationships between code entities (dependencies, impacts, modifications) are naturally expressed as graphs, enabling sophisticated queries for architectural insights.

**Why Entity-Observation Model?**: Flexible fact management without rigid schemas, allowing incremental knowledge building as analysis progresses.

**Why JSON Persistence?**: Analysis results stored as files enable review, debugging, and provide fallback if graph queries fail.

## Requirements

- Claude Code (with MCP support)
- Neo4j (local or remote instance)
- GitHub CLI (`gh`) authenticated
- Git repository with PR access
- Neo4j MCP server configured in Claude Code

## Future Enhancements

- Parallel file analysis for large PRs
- Automated test recommendation based on entity changes
- Dependency impact visualization
- Historical PR comparison
- Integration with CI/CD systems
