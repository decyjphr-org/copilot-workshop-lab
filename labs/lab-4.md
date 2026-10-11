## Lab 4: Custom agents

**Goal:** Harden the preloaded agent, build team agents in `common-agents`, use them from the viewer repo, add a personal agent, and design a multi-agent workflow.

**Repo:** `common-agents` · **Surface:** CLI, Copilot app, GitHub.com

### Exercise 4.1: Harden the `code-explainer` agent

The preloaded `code-explainer` agent says "Do not edit code", but it has no `tools` list, so it can use every tool, including edit and shell. Instructions ask for behavior; the `tools` list enforces it.

1. Open `~/projects/common-agents/.github/agents/code-explainer.agent.md` and look at three problems:

   * `tools` is commented out, so every tool is allowed.
   * The `description` is a persona ("You are an expert..."), not a statement of what the agent does and when to pick it. Copilot uses the description to choose agents.
   * The body tells the agent to use `#tool:vscode.mermaid-chat-features/renderMermaidDiagram`, which exists only in VS Code.

1. Fix the frontmatter:

   ```markdown
   ---
   name: code-explainer
   description: Explains an unfamiliar codebase, including its structure, architecture, domain model, build and test steps, and hotspots, without changing any code. Use when onboarding to a repository or asking how something works.
   argument-hint: A repository, folder, or file to explain, or a question about how the code works
   tools: ['read', 'search']
   ---
   ```

1. In the body, change the Mermaid line to:
   ```
   Visualize the architecture with Mermaid diagrams in fenced code blocks.
   ```
1. Commit and push. Restart the CLI. Because `~/.copilot/agents` links to this folder, the change applies everywhere.
1. In the viewer repo, run `/agent`, select **code-explainer**,
1. Ask:
   ```
   Explain how this app's scene, parts, and layout modules fit together.
   ```
1. Ask it to
   ```
   Fix any bugs you find.
   ```
1. It should not do any edits

> [!IMPORTANT]
> ✅ **Checkpoint:** The explanation includes a Mermaid diagram, and the agent refuses, or isn't able, to edit files when you ask it to fix bugs.

### Exercise 4.5: See personal instructions shape agent output

If you completed optional [Exercise 1.7](lab-1.md#-stretch-goal-exercise-17-add-personal-custom-instructions), this exercise reuses your personal instructions. Otherwise, add “Use TypeScript, make this a command-line program, and use concise plain language” to the prompts below. You can compare the output again after setting personal instructions.

1. Open [Copilot Chat on GitHub.com](https://github.com/copilot) in your personal space.
1. Ask:

   ```text
   How can I trace Agent and Sub agent calls in a Copilot session in VSCode.
   ```

1. Follow up with:

   ```text
   Show me an example code that will parse the log and create a mermaid diagram of the calls
   ```

> [!IMPORTANT]
> ✅ **Checkpoint:** The parser is a TypeScript command-line program, and the response follows your plain-language rules. If you completed optional Exercise 4.4, run it on your coordinator session log and paste the Mermaid output into a Markdown preview. Otherwise, ask Copilot to provide a small synthetic log containing one coordinator call and two subagent calls, document that input format, and run the parser on it; no coordinator setup is required.

### 🧯 **Troubleshooting (Lab 4)**

<details>
<summary>Show how to fix agent loading, duplication, and permission issues</summary>

* **Your agent doesn't show up.** Check that the file ends in `.agent.md`, that `ls -l ~/.copilot/agents` shows the link to `common-agents/.github/agents`, and that you restarted the CLI. For org agents, check that the file is on the default branch.
* **An agent is listed twice inside `common-agents`.** In that repo, the same files load as project agents and, through the link, as personal agents. The project copy takes precedence. Outside `common-agents`, each agent loads once.
* **A different agent runs than the one you expected.** A same-named agent at another level is shadowing yours. See the precedence experiment.
* **The agent does more than you allowed.** Check the `tools` list. It's an allowlist, and leaving it out allows every tool (as in the original `code-explainer`).

</details>

### 🚀 **Stretch goal:** Exercise 4.2: Create a `readme-specialist` team agent

<details>
<summary>Show how to build and test the README agent</summary>

1. Create `~/projects/common-agents/.github/agents/readme-specialist.agent.md`:

   ```markdown
   ---
   name: readme-specialist
   description: Creates and improves repository README documentation
   tools: ['read', 'search', 'edit']
   ---

   Focus exclusively on README and supporting documentation.
   Do not modify source code. Prefer concise, task-oriented examples.
   ```

1. Add a persona, a workflow, and clear boundaries below the frontmatter.
1. Test it on its own repo. Start `copilot` in `common-agents`, run `/agent`, choose **readme-specialist**, and ask it to `Write a README that explains how to use this repository's skills and agents`.
1. Commit and push, then restart the CLI. You don't need to link anything; the new file is already in the linked folder.
1. Start `copilot` in `iphone-duo-viewer`, select **readme-specialist**, and ask it to `Write a README for this project`.

> [!IMPORTANT]
> ✅ **Checkpoint:** The agent works in both repos, edits only documentation, and refuses to change source code.

> [!NOTE]
> Keep `tools` to the minimum set. `tools: []` doesn't mean "read-only". It removes every tool.

**Optional (organization owners): promote it to the organization.** Copy the file to `agents/readme-specialist.agent.md` in your organization's `.github-private` repository and commit it to the default branch. The agent then appears for everyone in the organization on GitHub.com, with no local setup.

</details>

### 🚀 **Stretch goal:** Exercise 4.3: Create a personal agent

<details>
<summary>Show how to create a personal review-style agent</summary>

A personal agent is just for you. You save it to `~/.copilot/agents/`, which is linked to `common-agents/.github/agents`. Before creating this personal agent, complete the optional **Keep personal agents out of the team repo** steps at the end of [Lab 0](lab-0.md), so the `my-` prefix matches the `.gitignore` rule and the file stays on your machine rather than being committed to the team repo.

1. Create `~/.copilot/agents/my-review-style.agent.md`:

   ```markdown
   ---
   name: my-review-style
   description: Reviews changes using my concise, fix-oriented format
   tools: ['read', 'search']
   ---

   Review only the requested changes. Every finding must include a
   specific suggested fix. Omit praise and general summaries.
   ```

1. Restart the CLI.
1. Confirm Git ignores the file:

   ```bash
   cd ~/projects/common-agents && git status --short .github/agents
   # No output for my-review-style.agent.md
   ```

   **Windows 11 Pro (Command Prompt):**

   ```cmd
   cd %USERPROFILE%\projects\common-agents && git status --short .github/agents
   REM No output for my-review-style.agent.md
   ```

1. In the viewer repo, run `/agent`, select **my-review-style**, and review the `test-review` branch if you completed optional [Exercise 2.5](lab-2.md#-stretch-goal-exercise-25-turn-the-review-prompt-into-a-shared-skill). Otherwise, ask it to review a committed viewer change against the repository instructions.

> [!IMPORTANT]
> ✅ **Checkpoint:** Every finding has a suggested fix, and there's no praise or summary.

**Precedence experiment:** In the viewer repo, create `.github/agents/my-review-style.agent.md` with the same name, but tell it to write long, detailed reviews. Restart the CLI and run it again. Which one ran?

> [!NOTE]
> The GitHub Docs disagree on this. The CLI configuration directory reference and the CLI command reference both say project agents (`.github/agents/`) take precedence over personal agents (`~/.copilot/agents/`). The CLI how-to for creating agents says the agent in your home directory wins. Record what your CLI version does, and give team and personal agents different names so it never matters.

For repository, organization, and enterprise agents, the repository agent overrides the organization agent, which overrides the enterprise agent:

```text
Enterprise:   .github-private/agents/reviewer.agent.md
Organization: .github-private/agents/reviewer.agent.md
Repository:   .github/agents/reviewer.agent.md  ← wins
```

</details>

### 🚀 **Stretch goal:** Exercise 4.4: Design a multi-agent handoff

<details>
<summary>Show how to plan a coordinated multi-agent workflow</summary>

1. Draw this workflow for a real viewer feature, such as "Add a hinge fold animation":

   ```text
   Coordinator
    ├─ Researcher ──┐
    ├─ Security  ───┼─ fan-out → consolidated plan
    └─ Architect ───┘
                           │ human approval
                           ▼
                      Implementer
                           ▼
                    Read-only Reviewer
   ```

1. For each stage, write the input and output contract (what it receives and what it must return).
1. Fan out only tasks that are independent.
1. Give each worker the minimum tools. For example, the Researcher and Reviewer get `read` and `search` only. Reuse your hardened `code-explainer` as the Researcher.
1. Create the agents in `common-agents/.github/agents/`:
   * `coordinator.agent.md`, whose `agents` field lists the workers it can delegate to.
   * One file for each worker. Set `user-invocable: false` to hide the workers from the picker; the coordinator can still delegate to them.
   * On the Implementer and Reviewer, add `include-custom-instructions: true`. Subagents don't follow the repository's custom instructions unless you add this.
1. Add a human approval step between the consolidated plan and the Implementer.
1. Commit, push, and restart the CLI.

> [!IMPORTANT]
> ✅ **Checkpoint:** Run the coordinator in the viewer repo. You see separate subagent runs for the fan-out, it stops for your approval before it edits anything, and the Implementer follows the viewer's `threejs.instructions.md`.

</details>

---

[← Previous: Lab 3](lab-3.md) · [README ↑](../README.md#own-your-copilot-hands-on-lab-exercises) · [Next: Lab 5 →](lab-5.md)
