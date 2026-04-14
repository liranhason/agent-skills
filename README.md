# HSN Claude Configuration

Personal Claude Code configuration repo for team sharing and cross-machine setup.

## What's Included

- **Commands**: `/ask` and `/plan` slash commands
- **Skills**: `plan-execution` skill (auto-activates when working with plans)
- **Settings Template**: Base configuration (customize for your setup)

## Installation

### New Machine Setup

```bash
# 1. Clone this repo
cd ~/git
git clone <your-repo-url> hsn-claude

# 2. Create .claude directory if it doesn't exist
mkdir -p ~/.claude

# 3. Copy/symlink files
cp -r ~/git/hsn-claude/commands ~/.claude/
cp -r ~/git/hsn-claude/skills ~/.claude/

# 4. (Optional) Copy settings template and customize
cp ~/git/hsn-claude/settings.template.json ~/.claude/settings.json
# Edit ~/.claude/settings.json with your preferences
```

### Alternative: Symlink Method

If you want changes to auto-sync:

```bash
# After cloning, symlink instead of copying
ln -s ~/git/hsn-claude/commands ~/.claude/commands
ln -s ~/git/hsn-claude/skills ~/.claude/skills
```

**Warning**: With symlinks, changes in your `.claude/` directory will modify the repo. Good for personal use, risky for team sharing.

## Usage

### `/ask` Command

Read-only Q&A mode. Claude answers questions without modifying any files.

```bash
/ask How does the auth system work?
/ask What's the difference between X and Y?
```

**Use cases:**
- Quick questions about the codebase
- Understanding architecture without changes
- Research mode

### `/plan` Command

Create and manage implementation plans for large features.

```bash
# Create a new plan
/plan Add user authentication system

# Check status of all plans
/plan status

# Continue with next task
/plan next

# Execute specific task
/plan next 2.1
```

**Plan workflow:**
1. `/plan <description>` - Creates a structured plan with milestones and tasks
2. `/plan next` - Execute the next available task
3. Plan auto-updates with progress and completion dates
4. Each task commits code + plan updates together

**Auto-execute entire plan with /loop:**

```bash
# Execute all tasks in a plan automatically
/loop /plan next --yes @plans/my-feature.plan.md

# Self-paced: Claude decides when to run next task
/loop /plan next --yes @plans/my-feature.plan.md
```

This will:
- Execute task 1.1, commit, mark complete
- Execute task 1.2, commit, mark complete
- Continue until all tasks are done
- Stop automatically when plan is complete

**Use cases:**
- Execute a fully-defined plan end-to-end
- Let Claude work through tasks while you're away
- Useful for well-scoped, low-risk implementation tasks

**Safety notes:**
- Review the plan carefully before looping
- Best for plans with clear acceptance criteria
- Can cancel anytime with `/stop` or by interrupting
- Each task commits separately (easy to revert if needed)

### `plan-execution` Skill

Auto-activates when working with plan files. Handles:
- Task dependency checking
- Progress tracking
- Verification against acceptance criteria
- Atomic commits per task

**Trigger phrases:**
- "continue the plan"
- "what's the plan status"
- "run task 2.1"
- References to `plans/*.plan.md` files

## Team Sharing

**Option 1: Direct Copy (Recommended)**

Team members clone and copy files to their `~/.claude/` directory. Each person maintains their own settings.

**Option 2: Fork Pattern**

1. Team lead maintains this repo
2. Team members fork it
3. Merge updates from upstream as needed
4. Each fork can have personal customizations

## File Structure

```
hsn-claude/
├── README.md                    # This file
├── settings.template.json       # Base settings (customize locally)
├── commands/
│   ├── ask.md                  # /ask command
│   └── plan.md                 # /plan command
├── skills/
│   └── plan-execution/
│       └── SKILL.md            # Plan execution skill
└── plans/
    └── examples/
        ├── add-feature.plan.md      # Minimal 2-milestone plan
        ├── api-refactor.plan.md     # Medium plan with diagram
        └── auth-system.plan.md      # Large 4-milestone plan
```

---

## Plan File Format

Plans live in `plans/*.plan.md` in your project repo (not this config repo). The `/plan` command creates and manages them.

### Anatomy of a Plan File

```markdown
# Plan: <Title>

| Field   | Value        |
|---------|--------------|
| Status  | draft        |   ← draft | in-progress | complete
| Created | 2026-04-10   |
| Ticket  | ENG-123      |   ← or N/A
| Branch  | feature/foo  |   ← or TBD

## Context
Why this work exists — the problem, the constraint, the goal.

## Architecture Decisions
Key choices made upfront so implementers don't re-litigate them mid-task.

## Diagrams (optional)
Mermaid flowcharts or sequence diagrams.

## Milestones Overview
Numbered list of major phases.

---

## Milestone 1: <Name>

### 1.1 [ ] Task title

- **Files:** path/to/file.py, path/to/other.py
- **What:** Step-by-step instructions for the implementer.
- **Acceptance:** How to verify this task is done (command to run, behavior to observe).
- **Dependencies:** None | 1.1, 2.3

### 1.2 [x] Completed task title *(completed 2026-04-11)*

...

## Verification

End-to-end checklist — commands to run, behaviors to confirm — after all milestones are done.
```

### Task Checkboxes

| Symbol | Meaning |
|--------|---------|
| `[ ]` | Not started |
| `[x]` | Complete (add `*(completed YYYY-MM-DD)*` inline) |

### Example: `/plan status` Output

```
📋 Plan: Add JWT Authentication System
Status: in-progress | Created: 2026-04-10

Progress: 5/14 tasks complete (36%)

## Milestones
✓ 1. Data Layer (3/3) — COMPLETE
  ✓ 1.1 Add User model with hashed password field
  ✓ 1.2 Add password hashing utilities
  ✓ 1.3 Add JWT creation and validation utilities

✓ 2. Auth Endpoints (2/3) — IN PROGRESS
  ✓ 2.1 Implement POST /api/auth/register
  ✓ 2.2 Implement POST /api/auth/login
  ◯ 2.3 Implement POST /api/auth/refresh and logout

◯ 3. Route Protection (0/2)
◯ 4. Frontend Integration (0/2)

Next task: 2.3 Implement POST /api/auth/refresh and logout
Files: src/routers/auth.py
Dependencies: 2.2
```

### Why This Format Is Powerful

1. **Resumability** — Stop mid-task, come back weeks later, pick up exactly where you left off. No mental overhead reconstructing context.
2. **Parallelism** — Dependencies are explicit. Tasks with no deps can run concurrently across people or agents.
3. **Auditability** — Every architectural decision is documented with the reasoning. Future maintainers understand *why*, not just *what*.
4. **Incremental progress** — A 50-task feature becomes 50 small, testable units. Each one shippable and verifiable.
5. **Collaboration** — Multiple people (or Claude agents) can work on independent milestones simultaneously without stepping on each other.

### Example Plans

See `plans/examples/` in this repo for annotated examples:

| Example | Complexity | What it shows |
|---------|-----------|---------------|
| [`add-feature.plan.md`](./plans/examples/add-feature.plan.md) | 2 milestones, 2 tasks | Minimal structure for a simple UI feature |
| [`api-refactor.plan.md`](./plans/examples/api-refactor.plan.md) | 3 milestones, 5 tasks | Dependency tracking, flowchart diagram, incremental migration |
| [`auth-system.plan.md`](./plans/examples/auth-system.plan.md) | 4 milestones, 9 tasks | Full-stack feature with parallel milestone tracks |

## Customization

### Adding Your Own Commands

1. Create a new `.md` file in `commands/`
2. Add frontmatter with `description` and `allowed-tools`
3. Write the command logic
4. Commit and push (or keep local)

### Adding Your Own Skills

1. Create a directory under `skills/`
2. Add `SKILL.md` with skill logic
3. Skills auto-load on Claude Code startup

### Settings Customization

Edit `~/.claude/settings.json` (NOT the template) to add:
- Environment variables (`env`)
- Model preferences (`model`)
- Hooks (see Claude Code docs)
- Status line customization

**Example settings.json:**

```json
{
  "model": "opus",
  "env": {
    "MY_API_KEY": "..."
  },
  "hooks": {
    "PreToolUse": [...]
  }
}
```

## What NOT to Commit

- `~/.claude/projects/` - User-specific memory
- `~/.claude/settings.json` - Personal settings (may contain secrets)
- `~/.claude/cache/` - Cache files
- `~/.claude/history.jsonl` - Command history

The `.gitignore` in this repo excludes these automatically.

## Updating

### Pull Latest Changes

```bash
cd ~/git/hsn-claude
git pull

# If you copied files (not symlinked):
cp -r commands ~/.claude/
cp -r skills ~/.claude/
```

### Contribute Changes

```bash
cd ~/git/hsn-claude

# Make changes to commands/ or skills/
# Test in your local .claude/ directory first

git add .
git commit -m "Add new command for X"
git push
```

## Troubleshooting

**Commands not showing up:**

```bash
# Verify files are in the right place
ls ~/.claude/commands/
# Should show: ask.md, plan.md

# Restart Claude Code or reload
```

**Skills not activating:**

```bash
# Check skill structure
ls ~/.claude/skills/plan-execution/
# Should show: SKILL.md

# Verify SKILL.md has proper frontmatter
```

**Settings not loading:**

```bash
# Verify JSON syntax
cat ~/.claude/settings.json | jq .

# Common issue: trailing commas (not valid JSON)
```

## References

- [Claude Code Documentation](https://claude.ai/code)
- [Custom Skills Guide](https://docs.anthropic.com/en/docs/claude-code/custom-skills)
- [Slash Commands](https://docs.anthropic.com/en/docs/claude-code/commands)

## License

MIT (or whatever you prefer)
