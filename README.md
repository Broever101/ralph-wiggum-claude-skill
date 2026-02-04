# Ralph Wiggum Claude Skill

> I got tired of pointing to online guides or explaining what Ralph Wiggum is over and over again. So I made this script. All you have to do is run the command, give it a task, and it converts that into a Ralph loop automatically.

A [Claude Code](https://claude.ai/code) skill that sets up autonomous iterative development loops using the Ralph Wiggum technique.

## What is Ralph Wiggum?

Ralph Wiggum runs Claude Code in a continuous autonomous loop, forcing it to keep working until tasks are truly complete. It solves the common problem of AI agents finishing work too early or declaring things "done" when they aren't.

**Best for:**
- Long-running tasks with clear goals
- Projects where you already have a PRD or detailed plan
- Tasks benefiting from continuous iteration
- Autonomous testing and development

**Not ideal for:**
- Exploratory work without clear goals
- Quick one-off tasks
- Situations needing frequent human input

## Installation

### Option 1: Clone to Claude Skills Directory (Recommended)

```bash
# Navigate to your Claude skills directory
cd ~/.claude/skills/

# Clone this repo
git clone https://github.com/Broever101/ralph-wiggum-claude-skill.git ralph-wiggum
```

Now you can invoke it with: `/ralph-setup`

### Option 2: Manual Install

1. Create `~/.claude/skills/ralph-wiggum/`
2. Copy `SKILL.md` to that directory
3. Restart Claude Code

## Quick Start

### 0. Disable Existing Ralph Plugin (Important)

If you have any existing Ralph Wiggum plugin installed in Claude Code, **disable it** before using this skill. The plugin method can conflict with the bash loop approach and cause issues. This skill uses a bash-based loop which is more reliable.

### 1. Enable Sandbox (Critical)

Create or edit `.claude/settings.json` in your project:

```json
{
  "permissions": {
    "allow": ["Bash(*)", "Read(*)", "Write(*)", "WebFetch(*)"],
    "defaultMode": "acceptEdits"
  },
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true
  }
}
```

Or simply run `/sandbox` in Claude Code.

### 2. Run the Skill

In your project directory:

```
/ralph-setup
```

This creates:
- `PROMPT.md` - Task specification for the loop
- `ralph.sh` - The loop launcher script
- `plan.md` - JSON task list with completion tracking
- `activity.md` - Progress log

### 3. Customize Your Task

Edit `plan.md` with your actual tasks and `PROMPT.md` with project details.

### 4. Run the Loop

```bash
chmod +x ralph.sh
./ralph.sh          # Default 50 iterations
./ralph.sh 100      # Custom max iterations
```

## How It Works

1. Claude reads `PROMPT.md` which references `plan.md` and `activity.md`
2. It picks the highest priority incomplete task
3. Executes the work autonomously (no permission prompts)
4. Verifies and updates `activity.md`
5. Marks task complete in `plan.md`
6. Commits to git
7. Loop repeats until all tasks pass or max iterations reached

## Requirements

- **Sandbox enabled** - Required for autonomous execution
- **Disable existing Ralph plugin** - Avoid conflicts with the bash loop method
- **Playwright MCP** - For visual verification (optional but recommended)
- **Clear goals** - Well-defined tasks in `plan.md`
- **Max iterations** - Always set to prevent runaway costs

## Templates

See the `/examples` directory for template files:
- `plan.md.template` - Task list structure
- `PROMPT.md.template` - Loop prompt template
- `.mcp.json.template` - Playwright MCP config

## Troubleshooting

| Issue | Solution |
|-------|----------|
| **Loop won't exit** | Make sure the completion promise in `PROMPT.md` matches the one in `ralph.sh` |
| Permission prompts | Enable sandbox and set `autoAllowBashIfSandboxed: true` |
| Port already in use | `lsof -ti :PORT \| xargs kill -9` |
| Agent gets stuck | Always set max iterations; check plan.md for ambiguity |
| Plugin conflicts | Disable any existing Ralph Wiggum plugin in Claude Code |

### Completion Promise Matching

The bash script (`ralph.sh`) checks for specific completion promises to exit. Make sure the promise in your `PROMPT.md` matches one of these defaults:

- `<promise>COMPLETE</promise>` (default)
- `<promise>DONE</promise>`
- `<promise>FINISHED</promise>`

Or if using a custom promise, update both `PROMPT.md` and the `COMPLETION_PROMISE` variable in `ralph.sh`.

## References

- [Original Ralph Wiggum Technique](https://ghuntley.com/ralph/)
- [Ralph Wiggum Guide](https://github.com/JeredBlu/guides/blob/main/Ralph_Wiggum_Guide.md)
- [Ralph Orchestrator](https://github.com/mikeyobrien/ralph-orchestrator)

## License

MIT - feel free to use and modify for your projects.

## Credits

- Original technique by [Geoff Huntley](https://ghuntley.com/ralph/)
- Guide by [JeredBlu](https://github.com/JeredBlu)
- This skill by [Broever101](https://github.com/Broever101)
