---
description: Generate comprehensive PR documentation using graph-based analysis
allowed-tools: [Bash, Read, Write, Task]
argument-hint: <pr-number> [repo-path]
---

# PR Documentation Generator

Generate comprehensive documentation for Pull Request $1 in repository ${2:-.}

## Workflow

### 1. Fetch PR Metadata

Execute: `gh pr view $1 --json title,body,number,files,author,createdAt,updatedAt,state,changedFiles,baseRefName,baseRefOid,headRefOid`

Store PR metadata and file list in `memory_bank/prContext.md`

### 2. Initialize Graph

Use neo4j-operations skill to create the PR entity in Neo4j:
- Entity name: `pr:$1`
- Include: title, author, number, state, timestamps

### 3. Analyze Files

For each file in the PR's changed files:

**File Analysis Strategy:**

**Threshold Policy**: PRs with fewer than 12 files will have all files analyzed; PRs with 12 or more files will use strategic sampling.

**Effective Count Calculation:**
- Count only files with substantive changes (exclude generated files, lock files, trivial updates)
- Examples of excluded files: `package-lock.json`, `*.min.js`, `yarn.lock`, auto-generated migrations
- If effective count < 12: Analyze all files
- If effective count ≥ 12: Sample strategically (high-impact files, breaking changes, core logic)

**Rationale:**
- Balance between thoroughness and efficiency
- Token budget optimization for large PRs
- Maintain quality insights while respecting context limits
- 12-file threshold provides sufficient coverage for most PRs

**Implementation:**
- If <12 files (effective): Delegate each to file-analyzer agent sequentially
- If ≥12 files (effective):
  - Identify high-impact files (largest changes, critical paths)
  - Sample representative files from each category (services, configs, tests)
  - Process in batches, update prContext.md after each batch

**Edge Cases:**
- PRs with 11 effective files + 5 lock files: Analyze all 11 effective files
- PRs with 20 files but only 10 substantive: Analyze all 10 substantive files
- PRs with 15+ high-impact files: Sample top 10-12 by impact score

**Delegation:**
Invoke file-analyzer agent with:
- file_path: {file}
- base_commit: {baseRefOid}
- head_commit: {headRefOid}
- pr_number: $1
- repo_dir: ${2:-.}

Wait for each agent to complete and return summary.

### 4. Query Graph for Insights

Use neo4j-memory-mcp tools to query the completed graph and extract insights from stored entities and observations.

**High-Impact Changes:**

Since impact levels and change details are stored as observations on Change entities (e.g., "Impact level: high", "Lines added: 42"), use the find_memories_by_name tool to retrieve the PR entity and traverse its relationships:

```
Call: mcp__mcp-neo4j-memory__find_memories_by_name
Parameters: names = ["pr:$1"]

Process:
1. Retrieve the PR entity and all connected File and Change entities
2. Filter in code for Change entities with observations containing "Impact level: high"
3. Extract changeType, linesAdded, linesRemoved from their observations
4. Compile list of high-impact file changes with their details
```

**Business Logic Modifications:**

Search for BusinessLogic entities related to the PR:

```
Call: mcp__mcp-neo4j-memory__search_memories
Parameters: query = "pr:$1 BusinessLogic"

Process:
1. Search returns BusinessLogic entities connected to the PR
2. Extract name, purpose, and impact observations from results
3. Document relationships and modifications
```

**Service Changes:**

Search for Service entities affected by the PR:

```
Call: mcp__mcp-neo4j-memory__search_memories
Parameters: query = "pr:$1 Service"

Process:
1. Search returns Service entities and related File entities
2. Extract service name, endpoints, and associated file paths from observations
3. Map service modifications and dependencies
```

### 5. Generate Documentation

**CRITICAL**: Read ALL JSON files from `memory_bank/analysisResults/*.json` to extract rich technical details.

Create `pr-$1-documentation.md` with:

#### Part 1: Executive Summary
- PR overview (title, author, description)
- Summary statistics (files changed, lines added/removed)
- High-level categorization from graph queries

#### Part 2: Technical Deep-Dive (File-by-File)

For each high-impact file (read from analysisResults/*.json):

**Required sections per file**:
- **File Path & Changes**: `{file_path}` (+X/-Y lines)
- **Purpose**: What this file does (from JSON `purpose` or `file_type`)
- **Breaking Changes**: List ALL breaking changes from JSON `breaking_changes` array
  - If none, state "No breaking changes"
  - If present, list each with before/after examples
- **Key Changes**: Extract from JSON `key_insights` or `key_changes` array
  - Method signature changes
  - Class/function additions/removals
  - Architectural modifications
- **Entities Modified**: From JSON `entities` array
  - Classes: List class names with their purposes
  - Functions: List function names with signatures if available
  - Dependencies: List added/removed dependencies
- **Impact Assessment**: From JSON `impact_assessment`
  - Impact level (high/medium/low)
  - Risk assessment
  - Affected systems/services
- **Business Logic**: If JSON contains `business_logic` entities, document them

**Example format**:
```markdown
### src/tools_library/implementations/gemini.py

**Changes**: +274/-192 lines | **Impact**: HIGH

**Purpose**: Core LLM client for Gemini models

**Breaking Changes**:
1. Removed `LlmClientInterface` inheritance
2. `send_job()` signature changed - removed `temperature`, `top_k`, `stream` parameters
3. Response type changed from `Dict` to `Pydantic BaseModel`

**Key Changes**:
- Migrated from VertexAI SDK to Google GenAI SDK
- Added `get_thoughts()` function for model introspection
- Enhanced error handling with structured logging

**Entities Modified**:
- **Classes**: GeminiLlmClient (refactored)
- **Functions**: send_job, send_msg_raw, __create_config (renamed from __create_model)
- **Dependencies Added**: google.genai, pydantic.BaseModel
- **Dependencies Removed**: vertexai, vertexai.generative_models

**Impact**: Affects all downstream LLM consumers. Migration required.
```

#### Part 3: Cross-Cutting Concerns
- Architectural insights (from graph queries)
- Service dependencies affected
- Testing recommendations
- Deployment considerations

**Validation Checklist** (ensure before finalizing):
- [ ] Breaking changes from all high-impact files are documented
- [ ] At least 3 specific technical details per file
- [ ] Business logic entities are mentioned
- [ ] Dependencies added/removed are listed
- [ ] Method/class level changes are specified

Update prContext.md phase to "complete"

## Output

Documentation saved to: `pr-$1-documentation.md`
