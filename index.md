---
title: Claude Code Project Playbook
---

# Claude Code Project Playbook
*A personal pre-build ritual using the 4Ds framework*

---

## How to use this playbook

Open this document at the start of every new project. Work through it in order:

- **Part 1** is one-time setup — do it once per machine, then skip it forever
- **Part 2** is per-project setup — run through this every time you start something new
- **Part 3** is your planning ritual — do this before writing any code or content
- **The Appendix** is your prompt library — copy from here throughout your project

---

## Part 1: One-Time Setup
*Do this once per machine*

### 1.1 Install Claude Code CLI

Open your terminal and run:

```bash
npm install -g @anthropic-ai/claude-code
```

Verify it worked:

```bash
claude --version
```

**Common gotchas:**
- If `npm` is not found, you need to install Node.js first — download it from nodejs.org
- If you see a permissions error, do not use `sudo`. Instead, fix your npm permissions with the steps below

**Permissions error fix (macOS)**

npm's global directory is owned by root by default. Fix it by pointing npm at a directory you own:

```bash
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.zshrc
source ~/.zshrc
```

Then re-run the install:

```bash
npm install -g @anthropic-ai/claude-code
```

This is a one-time fix — all future global npm installs will work without permission errors.

### 1.2 API Key Configuration

Claude Code needs an Anthropic API key to work.

**Step 1:** Get your key at console.anthropic.com → API Keys → Create Key

**Step 2:** Add it to your environment. Open your shell config file:

```bash
# For zsh (default on modern Macs)
open ~/.zshrc

# For bash
open ~/.bashrc
```

Add this line at the bottom:

```bash
export ANTHROPIC_API_KEY="your-key-here"
```

Save the file, then reload it:

```bash
source ~/.zshrc
```

**Step 3:** Verify it's working:

```bash
claude
```

You should see the Claude Code prompt. Type `exit` to leave.

### 1.3 Global Settings

Claude Code stores global settings in `~/.claude/settings.json`. You don't need to set this up manually — Claude Code creates it automatically. But there are a few useful defaults worth setting early.

Open Claude Code and run:

```
/config
```

This opens an interactive settings panel. Key things to consider:
- **Default model** — set to the model you use most
- **Permission mode** — start with the default (asks for approval on risky actions)

---

## Part 2: Per-Project Setup
*Do this at the start of every project*

### 2.1 Initialise Claude Code in Your Project

Navigate to your project folder and open Claude Code:

```bash
cd your-project-folder
claude
```

That's it. Claude Code is now running in the context of your project.

### 2.2 CLAUDE.md — What It Is, Where It Lives, and How to Set It Up

**What it is (plain English)**

CLAUDE.md is a text file that Claude Code reads automatically every time you start a session in your project. Think of it as a briefing document — it tells Claude who you are, what this project does, how you like to work, and what rules to follow. Without it, Claude starts every session knowing nothing about your project. With it, Claude starts every session already oriented.

**Where it lives**

| Location | Purpose |
|---|---|
| `~/.claude/CLAUDE.md` | Global — applies to every project on your machine |
| `[project-root]/CLAUDE.md` | Project-specific — most of your work goes here |
| `[subdirectory]/CLAUDE.md` | Scoped — for monorepos or when a subfolder needs different rules |

Start with the project-root file. Add a global one once you know what you want applied everywhere.

**Step 1: Open Claude Code in your project**

```bash
cd your-project-folder
claude
```

**Step 2: Run /init**

```
/init
```

Claude Code scans your codebase and generates a starter CLAUDE.md in your project root.

**Step 3: Open and edit the file**

Open it in your code editor:

```bash
code CLAUDE.md        # VS Code
cursor CLAUDE.md      # Cursor
open CLAUDE.md        # system default editor
```

It's plain Markdown — `#` for headings, `-` for bullet points. Edit and save like any file.

Alternatively, ask Claude to update it for you from inside a session:

```
Add a rule to CLAUDE.md: never modify the schema file directly
```

Claude will make the edit and show you the change to approve.

**Step 4: Review what /init generated**

Go through it section by section:

- **Project overview** — Is the description accurate? Fix anything wrong or vague
- **Commands** — Are the build/test/lint/run commands correct? Claude uses these
- **Conventions** — Did it pick up your actual coding style, or did it guess?
- **What Claude should/shouldn't do** — Usually blank after `/init`. This is the most valuable section to fill in yourself

**Step 5: Add what Claude couldn't infer**

- Why this project exists
- External services, APIs, or databases it connects to
- Anything Claude should never do
- Your preferences (e.g., "always ask before installing new dependencies")
- Domain knowledge a new developer would need on day one

**Step 6: Test it**

Exit and reopen Claude Code in the same project:

```bash
claude
```

Ask: *"What do you know about this project?"*

Claude's answer tells you exactly what landed and what's missing.

**Annotated example snippet:**

```markdown
# My Project

## What this is
A CLI tool that processes invoices from Xero and pushes them to a Google Sheet.
(Claude needs domain context to name things correctly and avoid breaking the Xero integration)

## Commands
- `npm run dev` — local dev server
- `npm test` — runs Jest suite
- `npm run build` — compiles to /dist

## Rules
- Never modify src/xero-client.ts directly — it's a generated file
- Always ask before adding new npm dependencies
```

### 2.3 Permissions — What Claude Can Do Without Asking You

**What permissions are (plain English)**

When Claude Code runs commands, edits files, or calls tools, it can either ask your approval each time or act automatically. Permissions are how you control that boundary. Set them too tight and Claude interrupts you constantly. Set them too loose and Claude does things you didn't intend. The goal is a calibrated middle ground.

**Where permissions live**

| File | Purpose |
|---|---|
| `~/.claude/settings.json` | Global — applies to every project |
| `[project-root]/.claude/settings.json` | Project-level — committed to the repo |
| `[project-root]/.claude/settings.local.json` | Local overrides — not committed, personal to your machine |

**How to set permissions**

**Option 1: During a session (easiest)**

When Claude asks permission to run something, you'll see options to approve once or always allow. Choosing "always allow" writes the permission to your settings automatically. This is the lowest-friction way to build up your permissions over time.

**Option 2: Edit settings.json directly**

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run test)",
      "Bash(npm run lint)",
      "Bash(git status)",
      "Bash(git diff*)"
    ],
    "deny": [
      "Bash(git push*)",
      "Bash(rm -rf*)"
    ]
  }
}
```

**Option 3: Use the /config command**

```
/config
```

Opens an interactive settings interface inside Claude Code.

**What to allow, gate, and block**

| Allow freely | Gate (approve each time) | Block entirely |
|---|---|---|
| Reading files | Installing packages | Force push to main |
| Running tests | Git commits | Dropping databases |
| Running linters | Git push | Production deploys |
| Git status/diff | Creating new files | `rm -rf` anything |
| Starting dev server | Editing config files | |

**The practical rule of thumb**

Allow anything that is read-only or easily reversible. Gate anything that changes shared state or is hard to undo. Block anything catastrophic.

**For a new project, start with this:**

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run*)",
      "Bash(git status)",
      "Bash(git diff*)",
      "Bash(git log*)"
    ]
  }
}
```

Then expand it as Claude asks for things and you decide they're safe to always allow.

### 2.4 MCP Servers — Giving Claude Extra Capabilities

**What MCP servers are (plain English)**

Out of the box, Claude Code can read and write files, run terminal commands, and talk to you. MCP servers are add-ons that give Claude new abilities — like browsing the web, reading your GitHub issues, querying a database, or searching Slack. Think of them as plugins. You install the ones relevant to your project and Claude can use them automatically.

MCP stands for Model Context Protocol — you don't need to understand the protocol, just know that it's the standard way to connect Claude to external tools.

**Where MCP servers are configured**

| File | Purpose |
|---|---|
| `~/.claude/settings.json` | Global — available in every project |
| `[project-root]/.claude/settings.json` | Project-level — only available in this project |

Put MCPs you use everywhere (like GitHub) in global settings. Put project-specific ones (like a database connection) in the project settings.

**How to add an MCP server**

**Option 1: Via the terminal (recommended)**

```bash
claude mcp add
```

This walks you through adding a server interactively.

For a specific server:

```bash
claude mcp add --name github -- npx -y @modelcontextprotocol/server-github
```

**Option 2: Edit settings.json directly**

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "your-token-here"
      }
    }
  }
}
```

**Option 3: Check what's already installed**

```
/mcp
```

Run this inside a Claude Code session to see all connected MCP servers and their status.

**MCP servers worth knowing about**

| Server | What it does | When to add it |
|---|---|---|
| GitHub | Read issues, PRs, repos | Any project on GitHub |
| Filesystem | Enhanced file operations | Large codebases |
| Puppeteer / Playwright | Control a browser | Web scraping, UI testing |
| PostgreSQL / SQLite | Query databases directly | Data projects |
| Slack | Read and send Slack messages | Team projects |
| Fetch / Web search | Browse the web | Research tasks |

**How to test that an MCP server is working**

After adding one, open Claude Code and ask:

```
What MCP tools do you have available?
```

Or try using it directly:

```
List the open issues in my GitHub repo
```

If Claude can answer, it's connected. If it says it doesn't have access, check your token or run `/mcp` to see the error.

**Practical note:** You don't need to add every MCP server upfront. Start with GitHub if your project lives there. Add others as the need arises.

### 2.5 Hooks — Automating What Happens Around Claude's Actions

**What hooks are (plain English)**

Hooks are commands that run automatically when Claude does something. For example: every time Claude edits a file, automatically run your linter. Every time Claude finishes a long task, send yourself a notification. Think of them as "if Claude does X, then automatically do Y" rules. You set them once and they run silently in the background.

**Where hooks are configured**

| File | Purpose |
|---|---|
| `~/.claude/settings.json` | Global — runs for every project |
| `[project-root]/.claude/settings.json` | Project-level — only runs in this project |

**The four hook events**

| Event | When it fires |
|---|---|
| `PreToolUse` | Just before Claude runs a tool |
| `PostToolUse` | Just after Claude runs a tool |
| `Notification` | When Claude sends you a notification |
| `Stop` | When Claude finishes responding |

**What hooks look like in settings.json**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npm run lint"
          }
        ]
      }
    ]
  }
}
```

**Practical hooks worth having**

**Auto-lint after file edits**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{ "type": "command", "command": "npm run lint" }]
      }
    ]
  }
}
```

*Keeps your code clean without having to remember to lint manually.*

**Auto-run tests after edits**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{ "type": "command", "command": "npm test" }]
      }
    ]
  }
}
```

*Catches breaking changes immediately while Claude is still in context.*

**Notify you when Claude finishes a long task**

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [{ "type": "command", "command": "osascript -e 'display notification \"Claude is done\" with title \"Claude Code\"'" }]
      }
    ]
  }
}
```

*Lets you walk away from long tasks without watching the screen.*

**How to test a hook is working**

After adding a hook, ask Claude to edit any file. Watch your terminal — if the hook fires you'll see the command output appear automatically. If nothing happens, check the command runs correctly on its own first, then double-check your JSON syntax — a missing comma or bracket will silently break it.

**Practical note:** Start with just the auto-lint hook if you have a linter set up. It's the highest-value, lowest-risk hook to have running.

---

## Part 3: The 4Ds Planning Ritual
*Do this before writing any code or content. This is your thinking work — the planning that makes everything that follows faster and better.*

### 3.1 Delegation — Deciding Who Does What

**What delegation means (plain English)**

Delegation isn't just "what can Claude do?" It's "what *should* Claude do, what should *I* do, and what should we do *together*?" Getting this wrong in either direction costs you time — over-delegating means Claude makes decisions that needed your judgment, under-delegating means you're doing work Claude could handle in seconds.

**The three delegation categories**

| Category | Description | Examples |
|---|---|---|
| Claude owns it | Claude does it, you review | Boilerplate code, formatting, research summaries, drafts, refactoring |
| You own it | You do it, Claude supports | Strategic decisions, creative vision, stakeholder judgment, ethical calls |
| Collaborative | You and Claude work through it together | Planning, debugging complex problems, iterating on drafts, making tradeoffs |

**How to think about each task**

Ask yourself four questions for each major task:

1. **Does this require context Claude doesn't have?** (Relationships, history, politics, intuition) → You own it
2. **Does this require speed and volume Claude is better at?** (Generating options, writing boilerplate, searching) → Claude owns it
3. **Does this require judgment calls you'd want to review?** → Collaborative
4. **What's the cost of Claude getting it wrong?** High cost → stay closer. Low cost → give Claude more autonomy.

**The delegation conversation — run this at the start of every project**

```
I'm starting a new project. Here's what it involves:

[describe your project in 3-5 sentences]

I want to plan how to delegate tasks between us before we start building.
Can you help me identify the major tasks this project involves, and then
we'll discuss each one — what you'd need from me, where I should stay
closely involved, and where you can move more autonomously?
```

**After Claude responds — push back**

```
What tasks on that list are you most likely to get wrong without
more context from me?
```

```
Are there any tasks where you'd recommend I stay more involved
than I might think I need to be?
```

```
What's the riskiest delegation decision in this plan?
```

**What to write down**

By the end of this conversation, you should have a simple list:

```
Task: Set up project scaffold          → Claude owns it
Task: Define core data models          → Collaborative
Task: Write API endpoints              → Claude owns it, I review
Task: Design error handling strategy   → I own it, Claude drafts options
Task: Write tests                      → Claude owns it
Task: Decide on deployment approach    → I own it
```

Save this. You'll refer back to it throughout the project.

**The one mistake to avoid**

Delegating a task to Claude and then not reviewing it. Delegation without review isn't delegation — it's abdication. The review is part of the work.

---

### 3.2 Description — Giving Claude What It Needs to Do Good Work

**What description means (plain English)**

Description is how you communicate a task to Claude. The quality of Claude's output is directly tied to the quality of your description. A vague prompt gets a generic answer. A precise prompt — one that gives Claude the right context, constraints, and success criteria — gets something genuinely useful.

**The five ingredients of a strong description**

| Ingredient | What it is | Example |
|---|---|---|
| **Role** | Who Claude is in this task | "You are helping me build a personal finance CLI tool" |
| **Task** | What you want done, specifically | "Write a function that parses CSV bank statements" |
| **Context** | Background Claude needs | "The CSV format comes from ANZ bank and has these columns..." |
| **Constraints** | What to avoid or limit | "Use only the standard library, no third-party packages" |
| **Success criteria** | What a good output looks like | "The function should handle missing values and return a typed object" |

**Weak prompt vs. strong prompt**

Weak:
```
Write a function to parse my CSV file
```

Strong:
```
I'm building a personal finance CLI tool in Python. I need a function
that parses CSV bank statements exported from ANZ bank.

The CSV has these columns: Date, Description, Debit, Credit, Balance.
Some rows have empty Debit or Credit fields — treat those as 0.

Use only the Python standard library. Return a list of typed dataclasses,
one per row. Include basic error handling for malformed rows — log the
error and skip the row rather than crashing.

A good result is a function I can call with a file path and get back
clean, usable data.
```

**The context Claude always needs at project start**

```
Here's the context for everything we'll work on together:

Project: [what you're building in 2-3 sentences]
Stack: [languages, frameworks, key tools]
Audience/users: [who this is for]
Constraints: [anything that limits our choices — budget, timeline, existing systems]
What success looks like: [how you'll know the project is done well]

Keep this context in mind for everything I ask you in this session.
```

**Describing tasks mid-project**

```
Task: [what you want]
Where we are: [what's already done that's relevant]
What I've already tried: [if applicable]
Output format: [code, list, prose, table — be specific]
```

**The one question to ask yourself before sending any prompt**

*"If Claude takes this completely literally and does exactly what I said, will I be happy with the result?"*

If the answer is no — add more constraints before sending.

**Common mistakes to avoid**

| Mistake | Fix |
|---|---|
| Describing the solution instead of the problem | Tell Claude what you need to achieve, not how to achieve it |
| Forgetting to specify output format | Always say what you want back |
| Leaving out constraints | If something is off-limits, say so explicitly |
| Assuming Claude remembers previous sessions | Re-establish context at the start of every session |
| Asking for too many things at once | One task per prompt for complex work |

---

### 3.3 Discernment — Evaluating What Claude Gives You

**What discernment means (plain English)**

Discernment is knowing when to trust Claude's output, when to verify it, and when to push back. Claude is fast and confident — but confidence isn't the same as correctness. Claude can be wrong in ways that sound completely right. Discernment is the skill of not being fooled by fluency.

**Calibrate trust based on output type**

| Output type | Trust level | How to verify |
|---|---|---|
| Boilerplate code structure | High | Quick read-through |
| Logic and algorithms | Medium | Test it, trace through edge cases |
| Facts and research | Low | Check primary sources |
| Library/API usage | Low | Check official docs |
| Security-sensitive code | Very low | Treat as untrusted until reviewed |
| Strategic recommendations | Medium | Apply your own judgment |
| Content about people or companies | Very low | Verify independently |

**The four questions to ask about any output**

1. **Does it actually do what I asked?** Read it against your original prompt.
2. **Does anything look off?** Trust your instincts.
3. **Did Claude make any assumptions?** Were they the right ones?
4. **What's the worst case if this is wrong?** Scale your review depth to the cost of a mistake.

**Prompts that help Claude self-critique**

```
Before I review this — what are the weakest parts of what you just
produced? Where are you least confident?
```

```
What assumptions did you make that I should verify?
```

```
What edge cases does this not handle?
```

```
If this is wrong, where is it most likely to be wrong?
```

**How to push back effectively**

Weak:
```
This doesn't look right
```

Strong:
```
The function returns an empty list when the input file has Windows-style
line endings. How would you fix that?
```

Or when you're not sure what's wrong:
```
I'm not confident this is correct. Walk me through your reasoning
step by step so I can identify where we might be going wrong.
```

**Red flags to watch for**

| Red flag | What it means |
|---|---|
| Claude cites a specific version number, URL, or date | Verify it — these are commonly hallucinated |
| The output is exactly what you hoped for | Healthy scepticism — did Claude tell you what you wanted to hear? |
| Claude uses a library or API method you don't recognise | Check the docs — it may not exist |
| Claude says "this should work" | That's a guess, not a guarantee — test it |
| The answer is very long and confident | Length and confidence don't equal correctness |

**The discernment rule for code**

Never merge code you haven't read. Claude can produce code that runs perfectly in the happy path and fails silently in edge cases.

```
Before I run this — give me a plain English summary of what each
function does and any edge cases I should test.
```

**Set your discernment standard upfront — add this to CLAUDE.md:**

```markdown
## Review standards
High scrutiny: security logic, data handling, anything user-facing
Medium scrutiny: core business logic, API integrations
Light scrutiny: scaffolding, formatting, boilerplate
```

---

### 3.4 Diligence — Maintaining Quality Throughout the Project

**What diligence means (plain English)**

Diligence is sustained attention to quality — not a one-time review at the end, but a consistent standard applied throughout. It's easy to move fast with Claude and accumulate small mistakes that compound into big problems. Diligence is how you prevent that.

**Set your done standard before you start**

```
This project is done when:
- [ ] The core feature works end-to-end with no known bugs
- [ ] Edge cases identified in planning are handled
- [ ] Code I'm not confident in has been reviewed or tested
- [ ] The output matches the vision I described in planning
- [ ] I'd be comfortable showing this to someone else
```

**The iteration loop**

For each task:

```
1. Describe the task clearly (3.2)
2. Claude produces output
3. You review it (3.3)
4. If it needs work — give specific feedback and iterate
5. If it's good — confirm it explicitly and move on
6. Note what was completed
```

**How to give feedback that improves the next iteration**

Vague:
```
This isn't quite right, try again
```

Specific:
```
Two things need fixing:
1. The error message on line 12 is too technical — rewrite it in plain English
2. The function doesn't handle empty input — add a guard clause at the top
```

**Check-in prompts to use throughout the project**

```
Summarise what we've completed so far and what's still outstanding
based on our original plan.
```

```
Are there any loose ends or unresolved decisions from our work today
that I should know about before I close this session?
```

```
Looking at what we've built so far — is there anything that feels
fragile or that you'd flag as needing more attention?
```

**Managing quality across sessions**

Claude doesn't remember previous sessions. Each time you reopen a project:

```
Here's where we left off: [paste your task list with completed items marked]
Today I want to focus on: [specific task]
Keep in mind: [anything important that came up last session]
```

**Knowing when you're actually done**

```
We've been working on [describe the task]. Before I mark this as done,
is there anything we haven't addressed that we should have? Any gaps,
assumptions, or risks I should review?
```

**Common diligence failures to avoid**

| Failure | What happens | Fix |
|---|---|---|
| No done standard | Scope creeps or ends prematurely | Write it before you start |
| Skipping session summaries | You lose track, Claude loses context | End every session with a summary prompt |
| Accepting "mostly right" | Small errors compound | Hold to your standard, iterate specifically |
| Not noting decisions made | You relitigate the same choices | Log key decisions in CLAUDE.md as you go |
| Reviewing too late | Problems are expensive to fix at the end | Review at the end of each task, not the project |

---

## Appendix: Prompt Library
*Organised by D. Copy, adapt, and make these yours over time.*

---

### Delegation Prompts

**Project kickoff — task breakdown**

```
I'm starting a new project. Here's what it involves:

[describe your project in 3-5 sentences]

Help me identify the major tasks this project needs. Then let's go
through each one and decide: what do you need from me, where should
I stay closely involved, and where can you move more autonomously?
```
*Forces the full task list to the surface before you start, not mid-build when changes are expensive.*

---

**Surfacing delegation risks**

```
What tasks on that list are you most likely to get wrong without
more context from me?
```
*Claude's first task list is usually optimistic. This prompt finds the gaps.*

---

**Clarifying ownership on a specific task**

```
For [specific task] — what would you need from me upfront to handle
this well without a lot of back and forth?
```
*Front-loads the information gathering instead of discovering gaps mid-task.*

---

### Description Prompts

**Session context setup — use at the start of every session**

```
Here's the context for everything we'll work on together today:

Project: [what you're building in 2-3 sentences]
Stack: [languages, frameworks, key tools]
Audience/users: [who this is for]
Constraints: [anything that limits our choices]
What success looks like: [how you'll know the project is done well]

Keep this in mind for everything I ask in this session.
```
*Prevents Claude from making assumptions that send the whole session in the wrong direction.*

---

**Individual task prompt — full structure**

```
Task: [what you want specifically]
Where we are: [what's already done that's relevant]
What I've already tried: [if applicable]
Constraints: [what's off limits]
Output format: [code / list / prose / table — be specific]
What good looks like: [success criteria]
```
*The output format and success criteria lines alone save a round of correction on almost every task.*

---

**Constraining an open-ended task**

```
Before you start — what clarifying questions do you have about this
task? I'd rather answer them now than correct the output later.
```
*Gives Claude permission to ask instead of assume.*

---

### Discernment Prompts

**Self-critique before you review**

```
Before I review this — what are the weakest parts of what you just
produced? Where are you least confident?
```
*Claude is often aware of its own uncertainty but won't volunteer it. This unlocks that.*

---

**Assumption check**

```
What assumptions did you make that I should verify?
```
*Fast way to find where Claude filled in gaps you didn't explicitly cover.*

---

**Edge case check — especially for code**

```
What edge cases does this not handle? What inputs would break it?
```
*Claude knows the failure modes of what it wrote. Make it tell you.*

---

**Pushback — when something looks wrong**

```
I'm not confident this is correct. Walk me through your reasoning
step by step so I can identify where we might be going wrong.
```
*Asking for reasoning surfaces the actual error instead of just producing a different wrong answer.*

---

**Verification prompt — for research and facts**

```
For any facts, version numbers, or API methods in your last response —
which ones should I verify against an external source before relying on them?
```
*Claude will tell you exactly which claims are most likely to be hallucinated.*

---

### Diligence Prompts

**End of session summary**

```
Summarise what we've completed today and what's still outstanding
based on our original plan.
```
*Forces a clean handoff from this session to the next. Paste the output into your next session opener.*

---

**Loose ends check**

```
Are there any loose ends, unresolved decisions, or things I should
know about before I close this session?
```
*Claude can see things in the work it produced that you might miss on a quick review.*

---

**Done check — before marking a task complete**

```
We've been working on [describe the task]. Before I mark this as done,
is there anything we haven't addressed that we should have? Any gaps,
assumptions, or risks I should review?
```
*Makes "done" a deliberate decision rather than a feeling.*

---

**Session opener — returning to an existing project**

```
Here's where we left off:
[paste your task list with completed items marked]

Today I want to focus on: [specific task]
Key decisions we've already made: [anything Claude shouldn't relitigate]
```
*Re-establishes context in under a minute. Claude has no memory between sessions.*

---

**Quality flag — mid-project check**

```
Looking at what we've built so far — is there anything that feels
fragile, inconsistent, or that you'd flag as needing more attention
before we keep going?
```
*Catches compounding small problems before they become expensive to fix.*
