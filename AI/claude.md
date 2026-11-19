### **Claude**:

https://code.claude.com/docs/en/quickstart

curl -fsSL https://claude.ai/install.sh | bash

!image.png

[]()

!image.png

#### Connecting Claude code to tools via MCP :

https://code.claude.com/docs/en/mcp

## **Claude Code:**

Claude Code is an agentic coding tool that understands your codebase, edits your files, runs commands, and integrates with your existing developer tools to help you get things done faster. It's available in your terminal, Visual Studio Code, the Claude Desktop app, on the web, and in JetBrains IDEs.

Unlike Claude.ai, Claude Code has direct access to your files, your terminal, and your entire codebase. Instead of copying and pasting code back and forth, it goes in and does the work itself.

The key differentiator is that Claude Code works as an **AI Agent**.

#### **What is an Agent?**

An AI Agent is software that can interact with its environment and perform actions to complete a defined goal. At its core, this works by having a large language model operating in a loop in real time. AI Agents can have access to tools, external services, or even other AI Agents to help reach their goals.

### **What Can Claude Code Actually Do?**

- **Read and understand your codebase.** You can ask Claude Code to explain a feature or trace a bug throughout your code.
- **Edit files across your project.** Claude Code can refactor a function and update every file that references it.
- **Run terminal commands.** It can execute your build script, run your tests, install packages, and use the output to decide what to do next.
- **Search the web.** If it needs documentation or the latest API references, it can look that up for you.

### **Using Claude Code Effectively**

To use Claude Code effectively, keep these three concepts in mind:

**The context window.** Think of this as Claude's working memory. It can hold a lot, but not everything at once. This is where the "agentic" aspect comes in — Claude finds strategic ways to locate answers within your codebase without loading the entire thing into context.

**You control its permissions.** Claude Code's permission modes let you choose how hands-on to be: it can ask you before running commands or making changes, or run in auto mode, where a background safety check screens each action instead. You're always in control, whether you prefer a hands-on or hands-off approach.

**It can make mistakes.** Just like any tool, Claude Code isn't perfect. It might misunderstand your intent, introduce a bug, or over-engineer a solution. Staying in the loop helps you catch these early.

### **The Agentic Loop**

Claude Code is best explained through the **agentic loop**:

1. You enter a prompt into Claude Code.
2. Claude gathers the context it needs by interacting with the model, which returns text or a tool call that Claude Code can execute.
3. It takes action — for example, editing a file or running a command.
4. It verifies the results and determines whether they achieve what your prompt set out to do.
5. If they do, Claude finishes and waits for the next prompt. If they don't, it loops back and tries again until the results are complete and verifiable.

Throughout this loop, you can add context, interrupt, or steer the model to help guide it toward your goal.

!image.png

### **Context**

Claude has a **context window** that determines how much of your conversation, file contents, command outputs, and more it can store and reference. Once you reach that limit, Claude Code compacts your conversation — automatically determining what it can remove or summarize to bring the context window back down to a usable size.

### **Tools**

Tools are the backbone of how agents work. Most AI assistants simply take text in and return text out. Tools let Claude Code determine *when* to execute code to get closer to completing a task. This could be a file-reading tool, a web search tool, or any number of other capabilities. Claude Code uses semantic understanding to determine when to call a tool and how to use the output.

### **Permissions**

Claude Code has several permission modes:

You can choose how much oversight to keep while Claude works. Press `Shift + Tab` to cycle between modes.

- **Manual:** Claude asks for explicit permission before editing a file or running a shell command.
- **Auto-accept:** Files are edited without asking, but commands still require approval.
- **Plan mode:** Uses read-only tools to compile a plan of action before starting any work.
- **Auto mode:** Claude works without permission prompts while a classifier checks each action in the background, targeted at blocking actions that are irreversible, destructive, or aimed outside your environment. When something is blocked, Claude usually finds a safer approach or asks you for the go-ahead.

All of this can be configured in your settings file. Be cautious when skipping permissions — giving Claude Code free rein to run commands means a mistake could be harder to catch before it happens.

Plan mode is great for planning complex changes or doing a safe code review. Many times you'll be asking Claude to handle multi-step implementations toward a feature, and this is exactly where Plan Mode excels.

### **Explore, Plan, Code, and Commit**.

Without following **Explore, Plan, Code, and Commit** workflow, most people jump straight to asking Claude to write code — which means more course-correcting later on.

Explore and Plan

The fastest way to handle the first two steps is with **Plan Mode**. In plan mode, Claude can't edit files — it just reads files to gather information about how it will tackle the implementation.

Claude will read relevant files, run some web searches, and give you a plan of action. Review it and decide if it meets your criteria. If not, ask it to revise specific areas. This is the best place to course-correct because it's before any code is written.

Code:

Once the plan looks good, select "approve" to accept it and let Claude work through the list items. Your permission mode determines how Claude proceeds: it can ask you at each step, auto-accept file edits, or run in auto mode with a background safety check screening each action.

A few tips to make the coding phase smoother:

- **Define a success criteria.** For Claude to be confident in its results, it needs to be clear on what "correct" looks like. Make this explicit when writing your plan.
- **Add tools.** Tools that help Claude complete its goals remove a lot of back and forth. For example, if you're building web UIs, install the Claude in Chrome extension so Claude Code can control a browser tab and test the UI directly.
    
    !The Claude in Chrome extension page in the Chrome Web Store
    
- **Include a test suite.** Give Claude a test suite it can continuously validate against. Claude can even write tests for you. Before handing this off, make sure the tests are a reliable source of truth to avoid false positives.

**Quick tip:** If you find Claude keeps running into the same issues, ask it to save the solution to its CLAUDE.md file.

Commit

Once you've tested the changes yourself and are happy with the results, it's time to push your code. Before you commit, run a **subagent code reviewer** to look at your work. A subagent gets a fresh pair of eyes on the codebase — it doesn't carry the bias the main agent might have from the session.

!image.png

Then get Claude to generate a commit message in your style.

### **What is the Context Window?**

Think of the context window as the amount of space Claude can hold in its memory. Whenever you enter a prompt, Claude reads a file, runs a tool call, or receives a tool call result, it's all adding to the context window. Since there's a finite amount of space, it becomes important to optimize how you use it.

#### **What Happens When Context Fills Up**

When you approach the limit, the context window is automatically **compacted**. Compaction summarizes important details and removes unnecessary tool call results to free up space. Note that this process can potentially lose details

You can run compaction manually with the `/compact` command. This compacts everything up to that point. It's handy when you want to free up context space while keeping a memory of what you previously worked on.

If you want to completely start from scratch with no memory of the previous session, run `/clear`. This removes everything.

To check the state of your context, run the `/context` command. You'll get a high-level overview of your context size, the categories taking up the most space, and a visual graphic showing the breakdown.

!image.png

A general rule of thumb:

- **Use `/compact`** when you're working on a specific feature and running up against the context limit but need to continue. Keeping the context relevant to your current feature is important.
- **Use `/clear`** when you want to start a new feature. You don't want the previous conversation to introduce bias into something new. For things you want Claude to remember across sessions, put them in your CLAUDE.md file so it doesn't have to rediscover things from scratch.

#### **Tips for Saving Context Space**

**Be specific.** A vague prompt might seem smaller, but it actually costs more context in the long run. Without clear instructions, Claude is forced to explore your codebase more and do its own reasoning — which takes up far more context space than a detailed prompt would.

**Manage your MCP servers.** MCP servers load all of their available tools into context by default, even when you're not using them. If you have servers configured for things unrelated to the current project, consider turning them off. You can also try "Skills," which work similarly to MCP servers but don't load everything into context upfront.

**Use subagents.** Subagents run in parallel with your main agent but have a completely separate context window. For tasks where you only need the answer — like "where are the authentication endpoints located?" — a subagent does the work and returns just a summary to your main agent, keeping your primary context clean.

## **Code review**

When you give Claude Code a task to complete in your codebase, Claude will often report back in a succinct way. Underneath the description of what Claude changed, there can be a variety of files that were changed (from small to major changes). It's a good practice to give every change a look yourself before you keep it, and then have Claude review it again from a clean context, without this session's history.

A diff is the before-and-after of a change: the lines removed and the lines added, file by file. The `/diff` command opens an interactive viewer of your uncommitted changes in that form, and it can also show what each of Claude's turns changed.

The most important things that deserve a second look every time are:

- **Changes you didn't ask for.** A config value that was edited while Claude was in the file or a rewritten helper method that you didn't mention.
- **Tests that got weaker.** If the project you're working in contains tests, identify any that were skipped, deleted, or loosened until they passed.
- **New packages and hard-coded values.** A dependency that was added for only one function, a URL or a key written straight into the code.

If the whole change is wrong, run `/rewind` (or press `Esc` twice on an empty prompt), pick the prompt that produced it, and choose **Restore code and conversation**. One limit worth knowing: files changed by shell commands Claude ran, such as a package install, aren't rolled back.

`/code-review` is that second reviewer: it reviews the change in a clean context, with none of your session's history, and reports what it finds. It edits nothing unless you ask it to.

### **The CLAUDE.md file**

One of the most useful features in Claude Code is the CLAUDE.md file. It gives Claude Code persistent memory about your project.

#### **The Problem It Solves**

When you open Claude Code without a CLAUDE.md file, it starts fresh every time. It has to re-explore your codebase, figure out what dependencies are needed, and understand what features are already implemented. Sometimes it makes assumptions, which makes it harder to steer Claude in the right direction.

CLAUDE.md solves this. It's a Markdown file you add to the root of your project, and Claude Code reads it automatically every time you start a session. Think of it as an onboarding script for your codebase. The contents of the CLAUDE.md file are appended to your prompt.

#### Example:

!image.png

It's straightforward. Now if you ask Claude Code to create a React component, it already knows to use Tailwind for styling and to follow your code conventions.

### **CLAUDE.md is for Teams**

You can (and should) commit your CLAUDE.md to version control so your team benefits from it. There's actually a hierarchy of memory files depending on who they're for:

- **Project-level CLAUDE.md** lives in the root directory of your project. Shared with the team.
- **User-level CLAUDE.md** lives in your configuration folder. This one is just for you and applies across all your projects. Put your personal preferences here.

### **Subagents**

Claude can delegate tasks to subagents that break them down and run component tasks in parallel, improving your context management. Each subagent operates in its own isolated context window.

#### **How It Works**

Managing context in Claude Code is important. A lot of the context window gets consumed by things like tool calls exploring your codebase or running web searches for research. What Claude discovers during that exploration isn't always relevant to the main feature you're developing.

This is where subagents come in. Claude spawns a subagent to handle a task like "explore this codebase for me." The subagent runs in parallel with its own context window, does all the exploration work, and once finished, summarizes its findings and returns that summary back to Claude.

The result: you get the answer you were looking for, without the entire journey it took to get there cluttering your main context.

Claude Code includes several built-in subagents that you can use immediately, like the general purpose subagent—use for multi-step tasks that require both exploration and action. The explore subagent—use for fast searching of code bases. The plan subagent—use during plan mode for research and analysis of your code base before presenting a plan.

#### **Creating Your Own Subagent**

Subagents are defined in Markdown files with YAML frontmatter. The easiest way to get started is to let Claude generate one for you. Run:

`/agents`

Then select "Create new agent." You'll walk through steps including choosing the scope of the agent, defining its purpose, selecting the tools it has access to, and even picking a color for it.

Claude will generate a name, description, and prompt for the subagent. This also tells Claude when to call the subagent based on the prompts you give it.

#### **Further Customization**

Subagents can be customized further. Here are some highlights:

- **Persistent memory** lets your subagent retain memory across conversations. This is great if you're using it consistently on the same projects.
- **Preload skills** into subagents by adding the `skills` key and listing skills by name. Note that unlike skills in your main conversation, the entire skill is loaded into context here.

### Skills

Every time you explain your team's coding standards to Claude, you're repeating yourself. Every PR review, you re-describe how you want feedback structured. Every commit message, you remind Claude of your preferred format. And skills fix this.

A skill is a markdown file that teaches Claude how to do something once, and Claude applies that knowledge automatically whenever it's relevant. Claude skills are folders of instructions, scripts, and resources that agents can discover and use to do things more accurately and efficiently.

With Claude Code, we have the SKILL.md file. The description is how Claude decides whether to use the skill. When you ask Claude to review this PR, it matches your request against available skill descriptions and finds this one. Claude reads your request, compares it to all available skill descriptions, and activates the ones that match.

You can store skills in a few places depending on who needs them. Personal skills go in the home directory .claude/skills, and follow you across all your projects. These are your preferences, your commit message style, your documentation format, how you like code explained. Project skills go in the .claude/skills inside of the root directory of your repository. Someone who clones the repository gets these skills automatically. This is where team standards live, like your company's brand guidelines, preferred fonts, and colors that you use for web design.

Claude Code has several ways to customize behavior. Skills are unique because they're automatic and task specific. CLAUDE.md files load into every conversation. If you want Claude to always use TypeScript strict mode, that goes in your CLAUDE.md file. Skills on the other hand load on demand when they match your request. It only loads in the name and description, so it doesn't fill up your entire context window. Your PR review checklist doesn't need to be in the context when you're debugging. It loads when you actually ask for a review.

Slash commands require you to type them, skills don't. Claude applies them when it recognizes the situation. These work best for specialized knowledge that applies to specific tasks. Code review standards your team follows, commit message formats that you prefer, brand guidelines of your organization. If you find yourself explaining the same thing to Claude repeatedly, well, that's a skill waiting to be written.

### **MCP**

Model Context Protocol (MCP) is an open standard that lets Claude Code connect to external tools and data sources. When you ask a question, Claude automatically understands when it should use those tools to better handle your query.

A lot of your context lives outside your codebase — in databases, productivity apps, or public repositories. MCP bridges that gap.

Tools give agents like Claude Code the ability to perform actions that help them complete tasks more effectively. This is different from typical AI, where you just get a text response back.

For example, if your team uses Linear for project management, you can add a Linear MCP server to bring in the details of your specific issues. If you need up-to-date documentation for a dependency, a docs MCP server like Context7 can provide that to Claude Code.

#### **Adding an MCP Server**

You can add MCP servers with the `claude mcp add` command. There are two main types:

- **HTTP servers** are for remote services. These are hosted by the service provider and connect over the network.
- **Stdio servers** are for local processes that run on your machine.

You can manage your servers with `/mcp` inside a Claude Code session to see what's connected, check status, and disable servers you don't need.

#### **Scoping Servers**

MCP servers can be scoped in three ways:

1. **Local** — only available in the current project, just for you.
2. **User** — available across all your projects.
3. **Project** — uses a `.mcp.json` file that you check into version control so anyone on the codebase gets the exact same servers automatically.

### **Hooks**

Hooks let you run commands at specific points in Claude Code's lifecycle. The key difference between hooks and everything else covered in this course is that hooks are **deterministic** — they always run.

#### **Why Use Hooks**

You can tell Claude in your CLAUDE.md to run Prettier after every file edit. Most of the time it will. But sometimes it won't. A hook makes it happen every single time, no exceptions.

Common use cases include:

- Auto-formatting after file edits
- Logging all executed commands for compliance
- Blocking dangerous operations like modifying production files
- Sending yourself notifications when Claude finishes a task

#### **How They Work**

Hooks are configured in your `settings.json`. You pick an event, optionally set a matcher for which tools it applies to, and provide a command to run. Some of the most common events are:

- **PreToolUse** — runs before a tool call
- **PostToolUse** — runs after a tool call completes
- **UserPromptSubmit** — runs when you submit a prompt, before Claude processes it
- **Stop** — runs when Claude finishes responding
- **Notification** — runs when Claude sends a notification

#### **A Practical Example**

The most common hook: auto-formatting after edits. Set a **PostToolUse** hook with a matcher of `"Edit|MultiEdit|Write"` so it fires whenever Claude modifies a file. The command checks the file extension and runs the appropriate formatter — Prettier for TypeScript, gofmt for Go, whatever your project uses.

#### **Blocking with PreToolUse**

PreToolUse hooks can **block tool calls** before they execute. Your hook receives the tool name and input as JSON on stdin. The exit code determines the behavior:

- **Exit code 0** — proceed normally.
- **Exit code 2** — block the action. The stderr message gets fed back to Claude as feedback so it knows why it was blocked and can adjust.
- **Any other exit code** — a non-blocking error that gets shown to you but doesn't stop anything.

This is how you enforce hard rules. Block writes to a production config directory. Block bash commands that contain `rm -rf`. Block commits to main. Whatever your team needs to be *guaranteed*, not suggested.

#### **Sharing Hooks with Your Team**

Hooks configured in `.claude/settings.json` are project-level and can be checked into your repo. This means your entire team gets the same hooks automatically. Use the `CLAUDE_PROJECT_DIR` environment variable in your commands to reference scripts stored in your project, so they work regardless of Claude's current working directory.