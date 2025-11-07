---
description: Guidelines for using the Neo4j memory server with PR documentation workflows
globs: **/pr-*.md, **/pr-documentation-generator.md
alwaysApply: false
---

# PR Documentation Neo4j Schema

This document defines the approved entity types and relation types that can be used with the Neo4j memory server via MCP tools for PR documentation workflows. Any new types must get approval from a user before adding and use.

## Entity Types

| Type | Description | Properties |
|------|-------------|------------|
| `PullRequest` | Represents a GitHub Pull Request | title, description, number, author, createdAt, updatedAt, state |
| `File` | Represents a file in the repository | path, fileType, content |
| `Change` | Represents changes made to a file in a PR | linesAdded, linesRemoved, changeType, impact |
| `Component` | Represents a logical component or module in the codebase | name, description, purpose |
| `Function` | Represents a function or method in the code | name, signature, purpose, complexity |
| `Class` | Represents a class in the code | name, properties, methods, purpose |
| `Issue` | Represents a GitHub Issue | title, description, number, author, createdAt, updatedAt, state |
| `User` | Represents a GitHub user | login, name, email |
| `BusinessLogic` | Represents business logic components | name, purpose, impact, rules, validation |
| `Configuration` | Represents configuration settings or files | name, parameters, values, environment |
| `Service` | Represents service components | name, endpoints, methods, authentication |
| `Dependency` | Represents project dependencies | name, version, type, source |
| `Schema` | Represents data models or schemas | name, fields, relationships, validation |
| `Infrastructure` | Represents infrastructure as code | name, resources, configurations, provider |
| `Pipeline` | Represents CI/CD pipeline components | name, stages, steps, triggers |
| `Concept` | Represents abstract concepts in the codebase | name, description, principles, patterns |

## Relation Types

| Type | Description | From | To |
|------|-------------|------|-----|
| `MODIFIES` | Indicates a PR modifies a file | PullRequest | File |
| `HAS_CHANGE` | Indicates a file has a specific change | File | Change |
| `CONTAINS` | Indicates a file contains a component/function/class | File | Component/Function/Class |
| `DEPENDS_ON` | Indicates a dependency relationship | Component/Function/Class | Component/Function/Class |
| `REFERENCES` | Indicates a reference relationship | Any | Any |
| `CREATED_BY` | Indicates creation authorship | PullRequest/Issue | User |
| `FIXES` | Indicates a PR fixes an issue | PullRequest | Issue |
| `IMPLEMENTS` | Indicates a PR implements a feature | PullRequest | Component |
| `INTRODUCES` | Indicates a PR introduces a new entity | PullRequest | Any |
| `MODIFIES_LOGIC` | Indicates a specific relation for business logic changes | PullRequest | BusinessLogic |
| `CONFIGURES` | Relates to configuration changes | PullRequest/Change | Configuration |
| `DEPENDS_ON_EXTERNAL` | Indicates external dependency relationship | Component/Service | Dependency |
| `IMPACTS` | Indicates one entity impacts another | Any | Any |
| `IMPLEMENTS_CONCEPT` | Shows implementation of a concept | Component/Function/Class | Concept |

## Usage Guidelines

- **Always Reference This Schema Document**:
  - Before using the Neo4j memory server, consult this document
  - Any deviation from these types requires explicit user approval

- **Approved Entity Types**:
  - Use only the entity types defined in this document
  - Each entity type has specific properties that should be included

- **Approved Relation Types**:
  - Use only the relation types defined in this document
  - Ensure relations connect appropriate entity types as specified in the schema

- **Adding New Types**:
  ```
  // ✅ DO: Get approval before using new types
  <ask_followup_question>
  <question>I'd like to add a new entity type 'CodeBlock' to represent specific sections of code. Would you approve this addition to the Neo4j schema?</question>
  </ask_followup_question>
  
  // ❌ DON'T: Use unapproved types without permission
  <use_mcp_tool>
  <server_name>mcp-neo4j-memory</server_name>
  <tool_name>create_entities</tool_name>
  <arguments>
  {
    "entities": [
      {
        "name": "block:login-validation",
        "type": "CodeBlock", // Unapproved type!
        "observations": [...]
      }
    ]
  }
  </arguments>
  </use_mcp_tool>
  ```

- **Naming Conventions**:
  - Use consistent prefixes for entity names:
    - `pr:` for PullRequest entities (e.g., `pr:123`)
    - `file:` for File entities (e.g., `file:src/main.js`)
    - `change:` for Change entities (e.g., `change:src/main.js`)
    - `component:` for Component entities (e.g., `component:auth-module`)
    - `function:` for Function entities (e.g., `function:validateUser`)
    - `class:` for Class entities (e.g., `class:UserController`)
    - `issue:` for Issue entities (e.g., `issue:456`)
    - `user:` for User entities (e.g., `user:johndoe`)
    - `bl:` for BusinessLogic entities (e.g., `bl:order-calculation`)
    - `config:` for Configuration entities (e.g., `config:api-settings`)
    - `service:` for Service entities (e.g., `service:authentication`)
    - `dep:` for Dependency entities (e.g., `dep:react-18.0.0`)
    - `schema:` for Schema entities (e.g., `schema:user-model`)
    - `infra:` for Infrastructure entities (e.g., `infra:web-server`)
    - `pipeline:` for Pipeline entities (e.g., `pipeline:deploy-prod`)
    - `concept:` for Concept entities (e.g., `concept:authentication`)

- **Documentation Updates**:
  - When a new type is approved, update this schema document immediately
  - Include a clear description, properties, and usage examples
  - Reference the updated schema in subsequent code

- **Schema Validation**:
  - Before submitting any PR that uses the Neo4j memory server, verify all entity and relation types against this schema
  - Include a comment in the PR referencing this document and confirming compliance

## Example Usage

```xml
<use_mcp_tool>
<server_name>mcp-neo4j-memory</server_name>
<tool_name>create_entities</tool_name>
<arguments>
{
  "entities": [
    {
      "name": "pr:123",
      "type": "PullRequest",
      "observations": [
        "Title: Fix bug in login flow",
        "Description: This PR fixes the login issue reported in #100",
        "Number: 123",
        "Author: johndoe",
        "Created at: 2023-01-01T12:00:00Z",
        "Updated at: 2023-01-02T12:00:00Z",
        "State: open"
      ]
    }
  ]
}
</arguments>
</use_mcp_tool>
```

```xml
<use_mcp_tool>
<server_name>mcp-neo4j-memory</server_name>
<tool_name>create_relations</tool_name>
<arguments>
{
  "relations": [
    {
      "source": "pr:123",
      "target": "file:src/login.js",
      "relationType": "MODIFIES"
    }
  ]
}
</arguments>
</use_mcp_tool>
