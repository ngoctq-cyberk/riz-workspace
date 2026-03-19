---
description: Delegate coding task to Claude Code CLI with context-optimized prompt
---

# Claude Code CLI Delegation Workflow

This workflow allows Antigravity to act as a "Pre-flight Controller" that prepares the context and prompt, then hands off the actual coding execution to the `claude` CLI tool.

## 1. Context Collection & Prompt Engineering

**Goal**: Create the perfect prompt for Claude.

1.  **Analyze Request**: Understand exactly what the user wants to build or fix.
2.  **Gather Context**:
    - Identify the architecture and dependent files.
    - Use `ls -R` or `find` to get file paths if needed.
    - Read key files using `view_file` to understand the current state.
3.  **Construct Optimized Prompt**:
    - Create a single, highly detailed prompt string.
    - **INCLUDE**:
      - Clear Task Objective.
      - **Absolute Paths** of all files to modify or read.
      - Key constraints (e.g., "Use Feature-Sliced Design", "Do not break existing tests").
      - Pertinent code snippets or context from your analysis.
    - _Tip_: The prompt should be self-contained so `claude` doesn't need to ask follow-up questions.

## 2. Execution (Delegate to Claude)

**Goal**: Run the Claude CLI with the optimized prompt.

1.  **Execute Command**:
    - Run the following command:
      ```bash
      claude "YOUR_OPTIMIZED_PROMPT_HERE"
      ```
    - _Note_: Ensure the prompt is escaped correctly for the shell.
    - _Note_: If the prompt is very long, consider writing it to a temporary file (e.g., `.gemini/tmp/prompt.txt`) and running `claude -p .gemini/tmp/prompt.txt` (if supported) or piping it `cat .gemini/tmp/prompt.txt | claude`. **Standard method**: Just passing the string argument is usually sufficient for most CLIs.

## 3. Verification & Reporting

**Goal**: Verify the work and report back.

1.  **Monitor Status**: Wait for the `claude` command to exit.
2.  **Review Changes**:
    - Read the command output to see what files were modified.
    - Briefly check the changes (using `view_file` or `git diff`) to ensure they match the intent.
3.  **Report**:
    - Inform the user that Claude has completed the task.
    - Summarize the changes made.
    - Ask if they want to verify or deploy.
