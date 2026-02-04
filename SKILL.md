---
name: ralph-loop
description: Set up Ralph Wiggum loop for autonomous iterative development
arguments:
  - name: max_iterations
    description: Maximum number of loop iterations (default: 50)
    required: false
  - name: completion_promise
    description: Custom promise tag for completion detection (default: COMPLETE)
    required: false
---

# Ralph Wiggum Loop Setup

**IMPORTANT: When this skill is invoked, your ONLY job is to SET UP the Ralph Loop infrastructure:**

1. Create `PROMPT.md` with the task specification
2. Create `activity.md` for progress tracking
3. Create `ralph.sh` (or equivalent) launcher script
4. Create `plan.md` with JSON task list
5. Create supporting documentation

**DO NOT:**
- Run the loop yourself
- Execute the task in the prompt
- Invoke any Ralph Loop plugin
- Do the work the loop is supposed to do

**SET UP ONLY. Then stop.**

---

You are setting up a Ralph Wiggum loop for autonomous iterative development. **Create the following files in the current working directory:**

## Context: Understanding Ralph Wiggum

**What is Ralph Wiggum?**
Ralph Wiggum runs Claude Code in a continuous autonomous loop, forcing it to keep working until tasks are truly complete. It solves the common problem of agents finishing too early.

**Best used for:**
- Long-running tasks with clear goals
- Projects where you already have a PRD/detailed plan
- Tasks benefiting from continuous iteration

**Not ideal for:**
- Exploratory work without clear goals
- Quick one-off tasks
- Situations needing frequent human input

**Critical Success Factors:**
1. **Sandbox is mandatory** - Enables autonomous execution without permission prompts (Boris Cherney, Anthropic)
2. **Feedback loops are essential** - Agent needs ways to verify its work (Playwright MCP or Claude for Chrome)
3. **Bash loop > Plugin** - Fresh context per iteration reduces hallucination risk
4. **Always set max iterations** - Prevents runaway costs

**Reference Guide:** https://github.com/JeredBlu/guides/blob/main/Ralph_Wiggum_Guide.md

---

## Step 0: Enable Sandbox (CRITICAL)

Before creating Ralph files, enable sandboxing for autonomous execution. Create or edit `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(*)",
      "Read(*)",
      "Write(*)",
      "WebFetch(*)"
    ],
    "defaultMode": "acceptEdits"
  },
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true
  }
}
```

Customize permissions based on your project needs. The key is `autoAllowBashIfSandboxed: true` which prevents permission prompts.

Alternatively, run `/sandbox` in Claude Code to enable interactively.

---

## Step 1: Create PROMPT.md

Create `PROMPT.md` with the following template. Replace the placeholder text with project-specific details:

```markdown
@plan.md @activity.md

We are [PROJECT GOAL - e.g., "testing and completing Phase 2 of the dashboard"].

**CRITICAL: AUTONOMOUS EXECUTION ENABLED**
- You have FULL permissions to execute ALL bash commands autonomously
- Run curl, npm, tests - everything yourself
- DO NOT ask for permission - just execute
- DO NOT say "manual verification required" - execute and verify yourself

**IMPORTANT: ALL SERVICES ARE ALREADY RUNNING**
- [List running services and ports, e.g., "Port 8000: API Service"]

**DO NOT START SERVICES** - they are already running. If you need to restart a service, first kill the existing process:
```bash
lsof -ti :<PORT> | xargs kill -9 2>/dev/null
```

**First read activity.md to see what was recently accomplished.**

**Then open plan.md and choose the SINGLE highest priority task where "passes" is false.**

**For TESTING tasks (category: "testing"):**
1. Run the test command yourself
2. Verify the result
3. Take screenshots if using Playwright MCP
4. Document what passed/failed in activity.md
5. Update that task's "passes" field from false to true
6. Make one git commit with clear message

**For FEATURE tasks (category: "feature"):**
1. Read the task description and all steps
2. **EXECUTE** all commands yourself (create files, run npm, install deps)
3. **VERIFY** everything works before moving on
4. Append a dated progress entry to activity.md
5. Update that task's "passes" field in plan.md from false to true
6. Make one git commit for that task only

**ONLY WORK ON A SINGLE TASK PER ITERATION.**

**DO NOT run git init, do not change git remotes, and do not push.**

When ALL tasks in plan.md have "passes": true, output exactly:

<promise>COMPLETION_PROMISE</promise>

---

## Testing Commands Reference

[Add project-specific test commands here]

## High Priority Tasks

[List high-priority tasks if needed]
```

## Step 2: Create ralph.sh

Create `ralph.sh` with the following content:

```bash
#!/bin/bash
# Ralph Wiggum Loop Runner
# Autonomous iterative development using Claude Code

set -e

MAX_ITERATIONS={{MAX_ITERATIONS:-50}}
COMPLETION_PROMISE="{{COMPLETION_PROMISE:-COMPLETE}}"
DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "🤖 Ralph Wiggum Loop - Iterative Development"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "Max Iterations: $MAX_ITERATIONS"
echo "Completion Promise: <$COMPLETION_PROMISE>"
echo "Working Directory: $DIR"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""

# Check if PROMPT.md exists
if [ ! -f "$DIR/PROMPT.md" ]; then
    echo "❌ Error: PROMPT.md not found in $DIR"
    exit 1
fi

cd "$DIR"

for ((i=1; i<=MAX_ITERATIONS; i++)); do
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo "🔄 Iteration $i / $MAX_ITERATIONS"
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo ""

  result=$(claude -p --permission-mode bypassPermissions "$(cat PROMPT.md)" --output-format text 2>&1) || true

  echo "$result"
  echo ""

  # Check for completion promise
  if [[ "$result" == *"<promise>$COMPLETION_PROMISE</promise>"* ]] || \
     [[ "$result" == *"<promise>COMPLETE</promise>"* ]] || \
     [[ "$result" == *"<promise>DONE</promise>"* ]] || \
     [[ "$result" == *"<promise>FINISHED</promise>"* ]]; then
    echo ""
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "✅ All tasks complete after $i iterations!"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    exit 0
  fi

  echo ""
  echo "--- End of iteration $i ---"
  echo ""
  sleep 1
done

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "⚠️  Reached max iterations ($MAX_ITERATIONS)"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "Check $DIR/activity.md for progress"
exit 1
```

Then run: `chmod +x ralph.sh`

## Step 3: Create plan.md

Create `plan.md` with a JSON task list. Replace with your actual tasks:

```markdown
# [Project Name] - Ralph Loop Task List

## Overview
[Brief project description]

**Success Gate:** [Define what "complete" means]

---

## Task List
```json
[
  {
    "category": "setup",
    "description": "Example: Initialize project",
    "steps": [
      "Create project directory",
      "Run initialization command",
      "Verify setup"
    ],
    "passes": false
  },
  {
    "category": "feature",
    "description": "Example: Implement core feature",
    "steps": [
      "Create component file",
      "Add API endpoint",
      "Test functionality"
    ],
    "passes": false
  },
  {
    "category": "testing",
    "description": "Example: Test all endpoints",
    "steps": [
      "Test GET endpoint",
      "Test POST endpoint",
      "Verify responses"
    ],
    "passes": false
  }
]
```

---

## Completion Criteria
- [ ] All tasks have "passes": true
- [ ] All tests pass
- [ ] Documentation complete

---

## Project Structure

```
[Add project structure diagram]
```

## Known Issues

[Document any known issues or workarounds]
```

## Step 4: Create activity.md

Create `activity.md` as an empty log file with header:

```markdown
# Activity Log - Ralph Wiggum Loop

This file tracks progress across Ralph loop iterations.

---

```

## Step 4.5: Set Up Feedback Loop (IMPORTANT)

The agent needs a way to verify its work visually.

### Playwright MCP (Recommended)
Create `.mcp.json` in project root:
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest", "--headless", "--output-dir", "."]
    }
  }
}
```
Create screenshots directory: `mkdir screenshots`

---

## Step 5: Verify Setup

After creating all files, verify the setup:

```bash
# Check files exist
ls -la PROMPT.md ralph.sh plan.md activity.md

# Make script executable
chmod +x ralph.sh

# Show instructions
echo ""
echo "✅ Ralph Wiggum loop setup complete!"
echo ""
echo "To start the loop, run:"
echo "  ./ralph.sh [max_iterations]"
echo ""
echo "Default: 50 iterations"
echo ""
```

## Running the Loop

After setup:
```bash
# Start the loop (default 50 iterations)
./ralph.sh

# Or specify max iterations
./ralph.sh 100
```

## Completion

The loop will automatically exit when:
1. All tasks in plan.md have `"passes": true`
2. Claude outputs `<promise>COMPLETION_PROMISE</promise>`

## Best Practices

1. **One Task Per Iteration**: Complete and verify one task before moving on
2. **Git Commits**: Commit after each task for history
3. **Test Everything**: Verify work before setting `passes: true`
4. **Document Findings**: Update activity.md with issues and solutions

## Troubleshooting

### Common Issues

**Agent gets stuck / infinite loop:**
- Ensure max iterations is set
- Check completion phrase is output correctly
- Review plan.md for ambiguous tasks

**Port already in use:**
```bash
lsof -ti :<PORT> | xargs kill -9 2>/dev/null
```

**Permission prompts blocking execution:**
- Verify sandbox is enabled in `.claude/settings.json`
- Check `autoAllowBashIfSandboxed: true`

**Expensive runs:**
- Always set max iterations (start with 10-20 for testing)
- Monitor activity.md and git commits for progress

---

## Template Variables

When invoking this skill, the following variables are replaced:
- `{{MAX_ITERATIONS}}` - Default: 50
- `{{COMPLETION_PROMISE}}` - Default: COMPLETE

## References

- Original technique: https://ghuntley.com/ralph/
- Ralph Orchestrator: https://github.com/mikeyobrien/ralph-orchestrator
- Guide: https://github.com/JeredBlu/guides/blob/main/Ralph_Wiggum_Guide.md
