This workflow generates comprehensive documentation for Pull Requests by analyzing the PR information, understanding the changes made, representing them as a graph, and generating documentation based on the insights.

This workflow follows these main steps:

1. Retrieve PR information using GitHub CLI
2. Analyze files before and after changes
3. Store information in a graph database using Neo4j MCP tools
4. Process all modified files with context management
5. Generate a graph representation of the PR changes
6. Query the graph for insights using advanced query strategies
7. Generate comprehensive documentation with categorized changes

Important Notes:

- This workflow is designed to be executed by an agent step by step
- Context management is critical due to potentially large file sizes
- The workflow uses MCP tools to interact with a Neo4j graph database
- Each step is implemented as a clear tool use, command execution, or agent output
- LLM-guided entity identification is used to recognize business logic, configuration, services, and other components
- The workflow leverages an extended Neo4j schema for more detailed PR insights
- the `gh` command is available to you and you are authenticated already
.github/pull_request_template.md
<inputs>
  pr_url:
    description: 'Pull Request URL'
      required: true
  repo_dir:
    description: 'Repository local directory root'
    required: true
  pr_number:
    description: 'Pull Request number'
    required: true
</inputs>

## Worktree Naming Convention

When analyzing PR files, agents may need to work with different versions of the codebase concurrently. Git worktrees provide an efficient way to have multiple working directories from the same repository without creating full clones.

### Naming Pattern Format

Use the following naming pattern for worktrees:

```
{repo-name}-pr{pr_number}-wt-{unique_identifier}
```

### Example

```
dst-python-ci-pr316-wt-file-analyzer-1
```

This indicates:
- Repository: `dst-python-ci`
- Pull Request: #316
- Purpose: worktree for file analyzer agent #1

### Rationale

1. **Uniqueness**: The pattern ensures each worktree has a distinct name, preventing conflicts when multiple agents work concurrently
2. **Traceability**: The name clearly identifies which PR and agent the worktree belongs to
3. **Context Clarity**: Anyone can immediately understand the purpose and scope of the worktree
4. **Easy Cleanup**: The consistent naming pattern makes it simple to identify and remove worktrees when work is complete

### When to Use Worktrees vs Clones

**Use Worktrees when:**
- Working within the same repository on different branches or commits
- Need to analyze multiple versions of files simultaneously
- Want to save disk space (worktrees share the same .git directory)
- Performing temporary analysis that will be cleaned up shortly

**Use Clones when:**
- Working with different repositories entirely
- Need complete isolation with separate git histories
- Long-term parallel development on different features

### Creating a Worktree

Parent agents should create worktrees before delegating to file-analyzer agents:

```bash
git worktree add ../{worktree_name} {branch_name}
```

Example:
```bash
git worktree add ../dst-python-ci-pr316-wt-file-analyzer-1 pr-316-branch
```

### Cleaning Up Worktrees

After analysis is complete, remove the worktree:

```bash
git worktree remove {worktree_name}
```

Or if the worktree directory was already deleted:

```bash
git worktree prune
```

### Important Notes

- **Parent Agent Responsibility**: Parent agents should create worktrees before delegation to ensure proper setup and avoid naming conflicts
- **Concurrent Work**: Each file-analyzer agent should have its own uniquely named worktree
- **Cleanup**: Always clean up worktrees after analysis is complete to avoid clutter and resource consumption
- **Location**: Worktrees should be created in a sibling directory (using `../`) to keep them organized and separate from the main repository

<detailed_sequence_of_steps>

# PR Documentation Generator - Detailed Sequence of Steps

## 1. Retrieve PR Information

1. Get the PR title, description, and other metadata:

    ```bash
    gh pr view ${inputs.pr_url} --json title,body,number,files,author,createdAt,updatedAt,state,changedFiles
    ```

2. Store PR information in the Neo4j memory server:

    ```xml
    <use_mcp_tool>
    <server_name>mcp-neo4j-memory</server_name>
    <tool_name>create_entities</tool_name>
    <arguments>
    {
      "entities": [
        {
          "name": "pr:${inputs.pr_number}",
          "type": "PullRequest",
          "observations": [
            "Title: ${pr_title}",
            "Description: ${pr_body}",
            "Number: ${pr_number}",
            "Author: ${pr_author}",
            "Created at: ${pr_created_at}",
            "Updated at: ${pr_updated_at}",
            "State: ${pr_state}",
            "Modified files count: ${modified_files_count}"
          ]
        }
      ]
    }
    </arguments>
    </use_mcp_tool>
    ```

3. Create file entities for each modified file that was in the returned data from the `gh` command:

    For each file:

    ```xml
    <use_mcp_tool>
    <server_name>mcp-neo4j-memory</server_name>
    <tool_name>create_entities</tool_name>
    <arguments>
    {
      "entities": [
        {
          "name": "file:${file_path}",
          "type": "File",
          "observations": [
            "Path: ${file_path}",
            "File type: $(echo "${file_path}" | awk -F. '{print $NF}')",
            "Modified in PR: ${pr_number}"
          ]
        }
      ]
    }
    </arguments>
    </use_mcp_tool>

    <use_mcp_tool>
    <server_name>mcp-neo4j-memory</server_name>
    <tool_name>create_relations</tool_name>
    <arguments>
    {
      "relations": [
        {
          "source": "pr:${inputs.pr_number}",
          "target": "file:${file_path}",
          "relationType": "MODIFIES"
        }
      ]
    }
    </arguments>
    </use_mcp_tool>
    ```

## 2. Analyze File Changes

1. Get the base commit and branch information of the PR:

    ```bash
    gh pr view ${inputs.pr_url} --json baseRefName,baseRefOid,headRefOid,title,body --jq '{baseRefName, baseRefOid, headRefOid, title, body}'
    ```

2. For each modified file, one at a time then loop until all done:

    1. Understand the file as it was before this PR changes:

        Read the content:

        ```bash
        cd $repo_dor && git show "$baseRefOid:$file_path"
        ```

        Please identify any entities and relationships in that file content and map to .clinerules/neo4j-schema-pr.md.
        Also make any insightful observations about the entity (free text).
        Analyse and store understanding in neo4j:

        ```xml
        <use_mcp_tool>
        <server_name>mcp-neo4j-memory</server_name>
        <tool_name>create_entities</tool_name>
        <arguments>
        [entities that you discovered from analysing the "before" file, with any observations. Must match the schema]
        </arguments>
        </use_mcp_tool>
        <use_mcp_tool>
        <server_name>mcp-neo4j-memory</server_name>
        <tool_name>create_relations</tool_name>
        <arguments>
         [relations that you discovered from analysing the "before" file. Must match the schema]
        </arguments>
        </use_mcp_tool>
        ```

    2. Retrieve and analyse the changes made to the content in the PR and identify key entities and relationships:

        Read the file changes:

        ```bash
        cd $repo_dor && git show "$base_commit..$head_commit" -- $file_path
        ```

        Please identify any entities and relationships in that file content and map to .clinerules/neo4j-schema-pr.md
        Also make any insightful observations about the entity (free text).

        For each identified entity, provide:
        - Entity type (e.g. BusinessLogic, Configuration, Service, Dependency, Schema, Infrastructure, Pipeline, Concept)
        - Entity name (descriptive identifier)
        - Purpose/function
        - Impact of changes
        - Relationships to other entities"

        Analyse and store understanding in neo4j:
  
        ```xml
        <use_mcp_tool>
        <server_name>mcp-neo4j-memory</server_name>
        <tool_name>create_entities</tool_name>
        <arguments>
        [identified entities with observations]
        </arguments>
        </use_mcp_tool>
        <use_mcp_tool>
        <server_name>mcp-neo4j-memory</server_name>
        <tool_name>create_relations</tool_name>
        <arguments>
        [identified relationships]
        </arguments>
        </use_mcp_tool>
        ```

## 3. Query Graph for Insights

1. Analyze the graph to identify patterns and insights:

    ```xml
    <use_mcp_tool>
    <server_name>mcp-neo4j-memory</server_name>
    <tool_name>search_memories</tool_name>
    <arguments>
    {
      "query": "What are the key changes in this PR? Include business logic, configuration, services, dependencies, schemas, infrastructure, and CI/CD pipeline changes."
    }
    </arguments>
    </use_mcp_tool>
    ```

2. Perform additional mcp neo4j tool call queries if you need to explore more:

    ```xml
    <use_mcp_tool>
    <server_name>mcp-neo4j-memory</server_name>
    <tool_name>[the tool you want to use]</tool_name>
    <arguments>
    [required arguments for the tool]
    </arguments>
    </use_mcp_tool>
    ```

3. OPTIONAL - If you need you can read the entire graph but **warning** it will use a lot of context:

    ```xml
    <use_mcp_tool>
    <server_name>mcp-neo4j-memory</server_name>
    <tool_name>read_graph</tool_name>
    <arguments>{}</arguments>
    </use_mcp_tool>
    ```

## 4. Generate Documentation

1. Generate comprehensive PR documentation based on the graph insights:

    ```xml
    <write_to_file>
    <path>"pr-${inputs.pr_number}-documentation.md"</path>
    <content>
    [PR Documentation structured as required by rules]
    </content>
    ```

</detailed_sequence_of_steps>
