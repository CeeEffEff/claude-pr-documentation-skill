# System Patterns

## Architecture Overview
The following directories are for configuring the assistant:
- `.clinerules` - Contains rule files and workflow definitions
  - `.clinerules/workflows/` - Contains workflow files for the assistant to execute

The PR Documentation Generator follows a sequential processing architecture:
1. PR Information Retrieval → 2. File Analysis → 3. Graph Representation → 4. Documentation Generation

## Design Patterns
- **Step-by-Step Workflow Pattern**: The PR Documentation Generator uses a clear, sequential workflow pattern where each step builds on the previous one.
- **Graph-Based Representation Pattern**: PR changes are represented as a graph with nodes (files, changes, PR) and relationships between them.
- **Context Management Pattern**: Explicit context management is implemented to handle large PRs with many files.
- **Continuation Task Pattern**: When context size approaches limits, the workflow creates continuation tasks with essential information preserved.
- **Schema Definition Pattern**: A standardized schema defines approved entity and relation types for the Neo4j memory server.
- **Workflow Structure Pattern**: All workflows follow a consistent structure defined in `.clinerules/workflow-creation.md`.
- **Input Parameter Pattern**: Input parameters are defined in a standardized format with descriptions and required flags.
- **Tool Invocation Pattern**: MCP tool invocations follow a consistent XML structure.
- **Command Execution Pattern**: Commands are executed using bash code blocks with consistent formatting.

## Component Interactions
- **PR Information Retrieval**: Uses GitHub CLI to fetch PR metadata and modified files with error handling
- **Neo4j Schema**: Defines standardized entity and relation types for the graph database
- **File Analysis**: Examines files before and after changes to understand modifications
- **Graph Representation**: Stores information in neo4j memory server using MCP tools following the schema
- **Documentation Generation**: Creates comprehensive documentation based on graph insights

## Critical Implementation Path
1. Create workflow file structure (Task 41) ✓
2. Implement PR information retrieval (Task 42) ✓
3. Develop file analysis capabilities (Tasks 43-44)
4. Create graph representation (Tasks 45-47)
5. Generate documentation (Task 48)
6. Integrate all components (Task 49)
