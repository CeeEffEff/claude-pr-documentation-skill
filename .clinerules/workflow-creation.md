---
description: Guidelines for creating and structuring Cline workflows
globs: **/*.md
alwaysApply: true
---

## Brief overview
This rule defines the structure, formatting, and best practices for creating Cline workflows. It ensures consistency across all workflow files and provides a clear template for new workflow development.

## Workflow structure
- All workflows must follow a consistent structure with these sections:
  - Header with workflow description and main steps
  - Important notes section
  - Inputs section with parameter definitions
  - Detailed sequence of steps
- Each workflow file must have a clear, descriptive title that indicates its purpose
- Include a brief summary of the main steps at the beginning of the file
- Add an "Important Notes" section highlighting key considerations for execution

## Input parameters
- Define all input parameters in a dedicated `<inputs>` section
- Format parameters consistently:
  ```
  <inputs>
    parameter_name:
      description: 'Clear description of the parameter'
      required: true/false
  </inputs>
  ```
- Mark parameters as `required: true` if they must be provided
- Use descriptive parameter names that clearly indicate their purpose
- Reference input parameters in the workflow using `${inputs.parameter_name}` syntax

## Tool invocations
- Use consistent XML structure for all MCP tool invocations:
  ```xml
  <use_mcp_tool>
  <server_name>server_name</server_name>
  <tool_name>tool_name</tool_name>
  <arguments>
  {
    "json_formatted_arguments": "values"
  }
  </arguments>
  </use_mcp_tool>
  ```
- Properly nest and close all XML tags
- Format JSON arguments with consistent indentation for readability
- Specify the correct server name for each tool (e.g., `mcp-neo4j-memory` for Neo4j operations)

## Command execution
- Enclose all commands in triple backticks with the appropriate language specifier:
  ```bash
  command --flag value
  ```
- Keep bash snippets small and focused, ideally one or two lines per step
- Avoid large bash scripts; instead, break operations into multiple small steps
- Let the markdown steps control the flow rather than complex bash logic
- Avoid unnecessary echo statements for error logs as commands will return errors naturally
- Reference variables using `${variable_name}` syntax
- Format commands for readability with proper spacing and line breaks
- Include comments to explain the purpose of complex commands

## Neo4j operations
- Follow the appropriate Neo4j schema based on your workflow type:
  - For PR documentation workflows: `.clinerules/neo4j-schema-pr.md`
  - For Terraform dependencies workflows: `.clinerules/neo4j-schema-terraform.md`
  - For common entity and relation types: `.clinerules/neo4j-schema-base.md`
- Use the appropriate Neo4j MCP tools for different operations:
  - `create_entities` for creating nodes
  - `create_relations` for creating relationships
  - `search_memories` for querying the graph
  - `read_graph` for retrieving the entire graph (use sparingly)
- Prefer direct Neo4j entity creation over temporary files for data storage
- Add entities to Neo4j as they are discovered rather than collecting them first
- Use consistent entity naming with appropriate prefixes (e.g., `pr:`, `file:`)
- Include all required properties for each entity type
- Create relationships only between existing entities
- Use approved relation types as defined in the schema

## Step organization
- Organize the workflow into clear, logical sections
- Use numbered headings for main sections (e.g., "## 1. Scan Directory")
- Include step-by-step instructions within each section
- Keep steps small and focused on a single operation
- Follow a consistent pattern for each step:
  1. Description of what the step does
  2. Command execution to gather data (keep commands simple)
  3. Data processing operations
  4. Transition to the next step
- Use clear transitions between sections
- Follow the example workflow structure closely (e.g., `.clinerules/workflows/pr-documentation-generator.md`)

## Documentation conventions
- Use consistent formatting throughout the workflow
- Include comments to explain complex operations
- Use proper indentation for readability
- Format code blocks with appropriate language specifiers
- Use descriptive variable names
- Document any conditional logic clearly
- Include examples where helpful

## Template usage
- Start new workflows by copying the template structure:
  ```
  # Workflow Title

  Brief description of the workflow's purpose.

  This workflow follows these main steps:
  1. Step 1
  2. Step 2
  3. Step 3

  Important Notes:
  - Note 1
  - Note 2

  <inputs>
    parameter_name:
      description: 'Parameter description'
      required: true
  </inputs>

  <detailed_sequence_of_steps>

  # Detailed Sequence of Steps

  ## 1. First Main Section
  
  1. Step description
     ```bash
     command
     ```
  
  2. Next step
     ```xml
     <use_mcp_tool>
     <server_name>server_name</server_name>
     <tool_name>tool_name</tool_name>
     <arguments>
     {
       "argument": "value"
     }
     </arguments>
     </use_mcp_tool>
     ```

  ## 2. Second Main Section
  
  [Additional steps]

  </detailed_sequence_of_steps>
  ```
- Adapt the template to the specific needs of the workflow
- Ensure all sections are properly formatted and complete
