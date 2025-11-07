# PR Documentation System

## Core Principles

When working with PR documentation workflows:

1. **Always check** `memory_bank/prContext.md` for current workflow state
2. **Use file-analyzer agent** for per-file analysis (never analyze files inline for PRs)
3. **Neo4j operations** go through mcp__mcp-neo4j-memory__* tools for consistency
4. **Save analysis results** to `memory_bank/analysisResults/{file_hash}.json` immediately
5. **Update prContext.md** with progress after each file
6. **CRITICAL: Read ALL JSON files** from `memory_bank/analysisResults/*.json` during documentation generation
7. **Extract rich details**: breaking changes, entities, dependencies, key insights from JSON
8. **Never summarize away** critical technical details like breaking changes or API modifications
9. **Generate documentation** only when all files have been analyzed

## Neo4j Schema

Reference: @memory_bank/graphSchema.md

Use consistent entity naming:
- `pr:{number}` for Pull Requests
- `file:{path}` for Files
- `bl:{name}` for Business Logic
- `config:{name}` for Configuration
- `service:{name}` for Services

## Context Management

- For PRs with >10 files: Use file-analyzer agent per file
- For PRs with ≤10 files: Can analyze inline if context permits
- Always delegate to agents when file analysis requires deep understanding

## MCP Tools Available

- Neo4j: `mcp__neo4j-memory__*` for graph operations
- IDE: `mcp__ide__*` for diagnostics and code execution
- file-analyzer agents must either clone into uniquely named repositories or use git worktrees which ar ealso uniquely named. Due to concurrent work and existing clones, the name should be unique and identify which agent created the clone/worktree. Alternatively, parent agents that delegate to file-analyzers should set these up before delegating.