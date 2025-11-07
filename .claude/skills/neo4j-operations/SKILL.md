---
name: neo4j-operations
description: Execute Neo4j graph database operations for PR analysis. Invoke when you need to create entities, create relationships, query the graph, or validate against the Neo4j schema. Auto-invoked for any graph data storage or retrieval.
allowed-tools: mcp__mcp-neo4j-memory__*
---

# Neo4j Operations Skill

You handle all interactions with the Neo4j graph database for PR documentation workflows.

## Capabilities

1. **Create Entities**: Store PR, File, Change, Function, Class, BusinessLogic, Configuration, Service entities
2. **Create Relationships**: Link entities with MODIFIES, HAS_CHANGE, CONTAINS, DEPENDS_ON, IMPLEMENTS, etc.
3. **Query Graph**: Retrieve architectural insights, dependencies, impact analysis
4. **Schema Validation**: Ensure all operations comply with @memory_bank/graphSchema.md

## Schema Reference

Always reference: @memory_bank/graphSchema.md

Use correct entity naming prefixes:
- `pr:{number}` for Pull Requests
- `file:{path}` for Files  
- `bl:{name}` for Business Logic
- `config:{name}` for Configuration

## Available MCP Tools

Use `mcp__mcp-neo4j-memory__create_entities` for creating entities
Use `mcp__mcp-neo4j-memory__create_relations` for creating relationships between **existing** neo4j entities
Use `mcp__mcp-neo4j-memory__search_memories` for querying graph data

## Common Operations

**Store PR Entity:**

Use `mcp__mcp-neo4j-memory__create_entities` to create a PullRequest entity with observations. This replaces the old Cypher approach with a structured entity model.

Example invocation:
```json
{
  "entities": [
    {
      "name": "pr:301",
      "type": "PullRequest",
      "observations": [
        "Title: Add dark mode toggle to application settings",
        "PR Number: 301",
        "Author: john-doe",
        "State: open",
        "Description: Implements dark mode theme switching with CSS variables"
      ]
    }
  ]
}
```

This creates a PullRequest entity named `pr:301` with rich observations about the PR details. The observations list stores all relevant metadata that was previously in separate Cypher properties.

**Link PR to File:**

Use `mcp__mcp-neo4j-memory__create_relations` to create relationships between existing entities. This replaces the old Cypher approach with a structured API that ensures both entities exist before creating the relationship.

Important: Both the PR and File entities must be created before establishing the MODIFIES relationship.

Example invocation:
```json
{
  "relations": [
    {
      "source": "pr:301",
      "target": "file:src/themes/darkMode.ts",
      "relationType": "MODIFIES"
    }
  ]
}
```

This creates a directed relationship from the PR entity to the File entity, indicating that the PR modifies the specified file. The tool validates that both entities exist before creating the relationship.

**Query File Changes:**

To retrieve all files modified by a PR and their associated changes, use `mcp__mcp-neo4j-memory__find_memories_by_name` to get the PR entity and traverse its relationships:

```
1. Call find_memories_by_name with the PR name (e.g., "pr:301")
2. The returned graph includes all related File entities via MODIFIES relationships
3. For each File, inspect its observations for change details
4. File observations include structured metadata like "Change type: {type}" and "Impact level: {level}"
5. Parse these observations to extract changeType and impact information
```

**Example Process:**

```json
{
  "step": "Find PR and retrieve related files",
  "tool": "mcp__mcp-neo4j-memory__find_memories_by_name",
  "input": {
    "names": ["pr:301"]
  },
  "result_processing": {
    "description": "The returned KnowledgeGraph contains the PR entity and all File entities connected via MODIFIES relationships",
    "extract_changes": "For each related File entity, read observations containing change metadata. Observations follow patterns like 'Change type: add', 'Impact level: high', or 'Lines modified: 45'"
  }
}
```

Alternatively, use `mcp__mcp-neo4j-memory__search_memories` to find files matching specific criteria:

```json
{
  "tool": "mcp__mcp-neo4j-memory__search_memories",
  "query": "pr:301 file impact high",
  "result_processing": "Returns matching entities including PRs, Files, and their relationships. Parse File observations to extract change details."
}
```

**Key Differences from Cypher:**
- Properties are stored as observations (text) rather than structured fields
- Change metadata must be parsed from observation strings
- The graph traversal happens implicitly through the returned relationships
- This approach ensures consistency with the entity-observation model used throughout the system

## Important

- Validate entity types against schema before creating
- Use parameterized queries to prevent injection
- Return structured results for easy parsing
