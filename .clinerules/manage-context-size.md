# Context Window Management

**⚠️ CRITICAL INSTRUCTIONS - YOU MUST FOLLOW THESE GUIDELINES ⚠️**
This guide provides **MANDATORY** instructions for effectively managing context size.

## ⚠️ CONTEXT WINDOW MONITORING - MANDATORY ACTION REQUIRED ⚠️

You **MUST** monitor the context window usage displayed in the environment details. When usage exceeds 50% of the available context window, you **MUST** initiate a task handoff using the `new_task` tool.

Example of context window usage over 50% with a 200K context window:

\`\`\`text

# Context Window Usage

105,000 / 200,000 tokens (53%)
Model: anthropic/claude-sonnet-4 (200K context window)
\`\`\`

**IMPORTANT**: When you see context window usage at or above 50%, you MUST:

1. Complete your current logical step
2. Use the `ask_followup_question` tool to offer creating a new task
3. If approved, use the `new_task` tool with comprehensive handoff instructions
