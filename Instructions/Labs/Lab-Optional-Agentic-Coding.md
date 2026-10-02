# Optional Lab - Agentic Coding

#### Duration: 45 minutes

### This lab requires GitHub Copilot Pro+ or Copilot Enterprise

## 🎯 Learning Objectives

By the end of this exercise, you will:

- Understand what GitHub Copilot Coding Agent is and how it differs from Agent mode in the IDE.
- Assign a task to the Coding Agent and monitor its progress.
- Review and iterate on the resulting pull request.
- Know when Coding Agent is the right tool versus local Agent mode.

## 🍎 Scenario: Delegate API Work to Copilot

The Daily Harvest team needs to ship improvements faster. Some tasks — adding an endpoint, writing tests for an existing module, updating docs — are well-scoped enough that you can hand them off entirely.

In this lab, you'll delegate one such task to the Copilot Coding Agent and follow it through the normal pull request flow.

## ⌨️ What Coding Agent Is

The Coding Agent works asynchronously from GitHub — not inside a chat session in your IDE. The workflow looks like this:

1. You create a GitHub Issue describing the task.
2. You assign the issue to Copilot.
3. Copilot creates a branch, implements the work, and opens a draft pull request.
4. You review, request changes, and merge — just like any other PR.

It's best suited for focused, well-defined tasks: bug fixes, small features, test additions, documentation updates. It requires a GitHub Copilot Pro+ or Enterprise subscription.

> **Note:** The Coding Agent runs on GitHub's infrastructure. You create the issue and review the PR locally (via `git pull`), but the implementation happens remotely. The steps below keep things as local as possible while working with a feature that is inherently tied to GitHub.

## 📝 Step 1: Create and Assign an Issue

**Instructions:**

1. Create an issue for a small, well-scoped task. You can do this from the CLI using the [GitHub CLI](https://cli.github.com/):

   ```bash
   gh issue create \
     --title "Add POST /api/contact endpoint" \
     --body "## Problem
   We need a contact form endpoint.

   ## Expected behavior
   - POST /api/contact accepts { name, email, message }
   - Returns 201 on success with { success: true }
   - Returns 400 if required fields are missing

   ## Acceptance criteria
   - Input validation for all fields (name, email, message required)
   - Email format validation
   - Request body size limited to prevent abuse
   - Unit tests covering happy path and validation errors"
   ```

2. Assign the issue to Copilot:

   ```bash
   gh issue edit <issue-number> --add-assignee @copilot
   ```

   If you prefer, you can also do both steps from the GitHub Issues page in your browser — the result is the same.

## 👀 Step 2: Monitor the Agent Session

**Instructions:**

1. Wait for Copilot to pick up the issue. It will create a branch and open a draft PR — this usually takes a few minutes.

2. Check the status from your terminal:

   ```bash
   gh pr list --author @me
   ```

3. Once the PR exists, fetch the branch locally to review the code in your IDE:

   ```bash
   git fetch origin
   git checkout <branch-name>
   ```

4. Look at what Copilot produced:
   - Does the PR description match the issue requirements?
   - Are the commits clean and focused?
   - Did it create tests?

## 🔍 Step 3: Review and Iterate

**Instructions:**

1. Review the code in your IDE just like you would for any teammate's contribution. Check:
   - Does the endpoint behave correctly?
   - Is the code readable and consistent with the rest of the project?
   - Are there tests, and do they cover the validation cases?

2. Run the tests locally:

   ```bash
   npm test
   ```

3. If you want changes, leave a review comment tagging `@copilot`:

   ```bash
   gh pr review <pr-number> --request-changes --body "@copilot Please add email format validation using a regex check"
   ```

4. Copilot will push updates to the same branch. Pull the changes and re-review:

   ```bash
   git pull origin <branch-name>
   ```

5. When you're satisfied, approve and merge:

   ```bash
   gh pr review <pr-number> --approve
   gh pr merge <pr-number> --squash
   ```

## 🎁 Optional Extensions

If time allows, try one or more of these:

- **Assign a second issue** — something like `PUT /api/cart/:id` — and compare how the Agent handles a different type of task.
- **Try the Agent Dashboard** at <https://github.com/copilot/agents> to see all active Coding Agent sessions.
- **From VS Code**, use the GitHub Issues integration and the **Assign to Copilot** option to start a Coding Agent task without leaving the IDE.
- **Compare with local Agent mode** — open Copilot Chat in Agent mode and ask it to implement the same task. How do the results differ? When would you pick one over the other?

## 🔐 Optional Security Track (GHAS + Copilot Autofix)

If your repository has GitHub Advanced Security enabled:

1. Enable the default CodeQL setup for the repository.
2. After a scan completes, check the findings:

   ```bash
   gh code-scanning alert list
   ```

3. Pick a finding and use **Copilot Autofix** (available in the Security tab) to generate a fix.
4. Review the suggested fix, commit it to a branch, and verify it locally.

## 🏆 Exercise Wrap-up

You completed an end-to-end delegated workflow:

1. Created and assigned an issue to the Coding Agent.
2. Monitored the autonomous implementation.
3. Reviewed and merged through standard PR practices — all from your terminal and IDE.

### Reflection Questions

1. Which task types are best suited for Coding Agent versus local Agent mode?
2. What review checks are non-negotiable before you'd merge a Coding Agent PR?
3. How should human review and AI-generated code complement each other in your team's workflow?
