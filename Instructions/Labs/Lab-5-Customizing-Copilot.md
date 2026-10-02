# Lab 5 - Customizing GitHub Copilot

#### Duration: 30 minutes

## 🎯 Learning Objectives

By the end of this lab, you should be able to:

- Understand what custom instructions are and why they matter for consistent Copilot output.
- Set up and enable custom instructions in your local IDE.
- Create an agent instructions file that governs how Copilot behaves in Agent Mode.
- Build reusable prompt files and custom chat modes.
- See firsthand how these customization layers work together.

## 🍎 Scenario: Taming inconsistency at The Daily Harvest

Your team relies on Copilot daily, but the results aren't always consistent. One person gets Jest tests, another gets Mocha. Someone gets CommonJS imports while the rest of the codebase uses ES modules. You keep repeating the same context in every chat session — the tech stack, the coding standards, the testing framework.

That's a waste of time. In this lab, you'll set up the customization files that solve this problem once and for all.

## 📄 What Are Custom Instructions?

Custom instructions are plain Markdown files that get prepended to every Copilot prompt behind the scenes. They let you define context, rules, and preferences so you don't have to repeat yourself.

There are three scopes, listed from lowest to highest priority:

1. **Organization** — applies to all repos in your GitHub org
2. **Repository** — lives in the repo, shared with the team
3. **Personal** — local to your machine, overrides everything else

This lab focuses on repository-level and personal instructions. When multiple levels exist, Copilot merges them with personal taking precedence over repository, which takes precedence over organization.

### A Quick Example

Say you put this in `.github/copilot-instructions.md`:

```md
Always use TypeScript with strict mode enabled.
Use Jest for all test files.
Prefer async/await over raw Promises.
```

Now every Copilot response in this repo will follow those rules without you having to mention them each time. Simple as that.

## ⚙️ Step 1: Enable Custom Instructions

Before any of this works, you need to flip a switch in your IDE settings.

**VS Code:**

1. Open Settings with `Cmd`+`,` (Mac) or `Ctrl`+`,` (Windows/Linux).
2. Search for `copilot instructions`.
3. Make sure **GitHub Copilot: Use Instruction Files** is checked.

**JetBrains IDEs (IntelliJ, WebStorm, etc.):**

1. Open Settings/Preferences.
2. Go to **Tools** → **GitHub Copilot**.
3. Check **Use Instruction Files**.

That's it. Copilot will now pick up any instruction files it finds in the repository.

## 📝 Step 2: Generate a Custom Instructions File

Rather than writing an instructions file from scratch, let Copilot do the heavy lifting.

1. Open GitHub Copilot Chat and switch to **Agent Mode**.
2. Click the **cogwheel** (⚙️) in the upper-right corner of the Chat window.
3. Select **Generate Agent Instructions** from the dropdown.
4. Copilot will analyze the repository and produce a `.github/copilot-instructions.md` file tailored to your project.
5. Read through it carefully. Does it match your actual tech stack? Did it pick up the right test framework? Adjust anything that looks off.

Take a minute to compare what Copilot generated against the instruction templates in `CoPilot-rules-template/instructions/`. You'll notice the templates cover security, code quality, and organization standards — these are the kinds of rules that belong in instruction files.

## 🤖 Step 3: Create an Agent Instructions File

Copilot's Agent Mode can run terminal commands, edit multiple files, and chain together complex workflows. That's powerful, but it also means you want some guardrails in place.

The `agents.instructions.md` file defines how Copilot should behave when working autonomously. Things like:

- What it should explain before making changes
- What commands it should never run without asking
- How it should handle errors
- Rules for installing dependencies

There's already a template ready for you at `CoPilot-rules-template/instructions/agents.instructions.md`. Let's put it to use.

**Instructions:**

1. Open and read through `CoPilot-rules-template/instructions/agents.instructions.md`. Pay attention to the six sections it covers: autonomy boundaries, tool usage, code generation, error handling, MCP servers, and file management.
2. Copy the file into your project's `.github/instructions/` directory (create the directory if it doesn't exist):
   ```
   mkdir -p .github/instructions
   cp CoPilot-rules-template/instructions/agents.instructions.md .github/instructions/
   ```
3. Open the copied file and customize it for your project. For example, you might want to:
   - Add project-specific commands that should always require confirmation
   - Specify which MCP servers are approved for use
   - Define cleanup rules for temporary files
4. Test it out: switch to Agent Mode and ask Copilot to perform a multi-step task (like adding a new endpoint with tests). Watch whether it follows the rules you defined — does it explain its plan first? Does it run linting after making changes?

<details>
<summary>Sample prompt to test your agent instructions</summary>

```
Add a new GET endpoint /api/health that returns { status: "ok" }.
Include a test file for it. Make sure everything passes linting.
```

Watch how Copilot approaches this. With the agent instructions in place, it should:
- Explain what it plans to do before starting
- Create the endpoint and test file
- Run lint and format checks afterward

</details>

## 📋 Step 4: Reusable Prompt Files

Instructions files set global rules. Prompt files solve a different problem: they let you package up a specific, reusable prompt that you or your team can invoke by name.

Prompt files live in `.github/prompts/` and use a YAML header plus a Markdown body. They can specify:

- **mode** — which chat mode to use (e.g., `agent`, `ask`)
- **description** — a short summary of what the prompt does
- **tools** — which tools the prompt should have access to

You can also use variables to make prompts dynamic:

| Variable | What it does |
|----------|-------------|
| `${workspaceFolder}` | Path to the workspace root |
| `${file}` | Path to the currently open file |
| `${selection}` | Currently selected text |
| `${input:name}` | Asks the user for input at runtime |
| `${input:name:placeholder}` | Same, but with placeholder text |

**Instructions:**

1. First, enable prompt files in your IDE:
   - **VS Code:** Open Settings, search for `Prompt Files`, and enable **Chat: Prompt Files**.
   - **JetBrains:** This is typically enabled by default when instruction files are enabled.
2. Create the prompts directory: `mkdir -p .github/prompts`
3. Create a new file `.github/prompts/explain.prompt.md` with the following content:

<details>
<summary>Example prompt file</summary>

```md
---
mode: 'ask'
description: 'Break down a code snippet in plain language'
---

Explain the following code clearly and concisely:

Code: ${input:code:Paste your code here}
Audience: ${input:audience:Who is this for? (e.g., beginners, senior devs)}

Include:
- What the code does at a high level
- A walkthrough of the key parts
- Any important concepts or patterns used
- A simple usage example
- When you'd use this approach in practice

Keep the language straightforward. Skip the jargon where possible.
```

</details>

4. To use it, open Copilot Chat and type `/explain`, then hit Enter.
5. Fill in the inputs when prompted and see how Copilot follows your template.

Try creating a second prompt file for something your team does often — maybe a `/review` prompt that checks code against your security guidelines, or a `/test` prompt that generates test cases for a given function.

## 🎨 Step 5: Custom Chat Modes

Copilot ships with **Ask** and **Agent** modes out of the box. Custom chat modes let you define your own specialized modes with specific tools and behaviors.

Chat mode files use the `.chatmode.md` extension and can live in `.github/chatmodes/` (shared with the team) or in your personal settings directory (just for you).

The YAML header supports:

- **description** — what this mode is for
- **tools** — which tools are available (e.g., `fetch`, `search`, `githubRepo`, `usages`)

**Instructions:**

1. Create the directory: `mkdir -p .github/chatmodes`
2. Create a file `.github/chatmodes/Plan.chatmode.md`:

<details>
<summary>Example chat mode file</summary>

```md
---
description: Think through an implementation plan without writing any code.
tools: ["fetch", "githubRepo", "search", "usages"]
---

# Planning Mode

You are in planning mode. Your job is to think through how to implement a feature or refactor existing code — but you must not write or edit any code.

Produce a Markdown document with these sections:

- **Overview**: What are we building or changing, and why?
- **Requirements**: What needs to be true when we're done?
- **Implementation Steps**: A detailed, ordered list of what to do.
- **Edge Cases**: Anything tricky that needs special attention.
- **Testing**: What tests need to exist to verify the work?

Be specific. Reference actual files and functions from the codebase where relevant.
```

</details>

3. To use the mode, open Copilot Chat. At the bottom of the chat input area, click on the current mode name and select **Plan** from the list.
4. Try it out: ask it to plan a feature and see how the response differs from a regular Agent Mode response. It should give you a structured plan without touching any files.

## 🏆 Wrapping Up

Here's what you set up in this lab:

| File | Purpose | Location |
|------|---------|----------|
| `copilot-instructions.md` | Global rules for every Copilot interaction | `.github/` |
| `agents.instructions.md` | Rules for Agent Mode behavior | `.github/instructions/` |
| `*.prompt.md` | Reusable, parameterized prompts | `.github/prompts/` |
| `*.chatmode.md` | Custom interaction modes | `.github/chatmodes/` |

These layers stack: instructions set the baseline, agent instructions add autonomy guardrails, prompt files package common workflows, and chat modes define entirely different interaction styles.

### Worth Thinking About

- How would [path-specific instruction files](https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot) (scoped to certain directories or file types) help in a monorepo with both frontend and backend code?
- Which prompts does your team repeat most often? Those are candidates for prompt files.
- Could a "Security Review" chat mode help catch issues earlier in development?

### Key Takeaways

- Custom instructions eliminate the need to repeat context in every chat session.
- Agent instructions give you control over what Copilot does when it's working autonomously.
- Prompt files and chat modes turn your team's best practices into reusable, shareable tools.
- All of these files are just Markdown — easy to version, review, and iterate on.

## 🔮 What's Next?

In Lab 6, you'll apply Secure by Design principles to your GitHub Copilot workflow.
